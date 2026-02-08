# Plan: Web Content Extraction Overhaul

## Problem

Smaug fetches article URLs from bookmarked tweets but stores **raw HTML** -- scripts, navigation, ads, styles, everything. Content is double-truncated (50KB at fetch, 10KB at storage), wasting 70-90% of storage on HTML boilerplate. The AI model receives garbage instead of readable article content.

## Solution

Replace the raw HTML fetch-and-truncate approach with a structured extraction pipeline that produces **clean markdown** and rich metadata using content extraction libraries.

---

## New Dependencies

| Package | Size | Purpose |
|---------|------|---------|
| `defuddle` | ~1.1MB | Primary content extractor (built-in markdown output, rich metadata) |
| `jsdom` | ~13MB | DOM implementation required by Defuddle (Node.js) and Readability |
| `@mozilla/readability` | ~155KB | Fallback extractor (Firefox Reader Mode algorithm) |
| `html-to-text` | ~175KB | Last-resort converter (strips all HTML tags) |

Going from 1 to 5 production dependencies. Justified by the magnitude of improvement: article content goes from garbage HTML to clean readable markdown.

---

## Extraction Pipeline

```
URL
 |
 v
fetch() -- 500KB HTML limit (up from 50KB, extractors need full page)
 |
 v  Check Content-Type: skip PDF/binary, proceed for text/html
 |
 v
Defuddle (primary) -- await Defuddle(html, url, { markdown: true })
 |  Returns: content (markdown), title, author, published, description,
 |           domain, site, wordCount, schemaOrgData, metaTags
 |
 +-- content.length > 100? --> return (quality: high)
 |
 v
Readability (fallback) -- new Readability(dom.window.document).parse()
 |  Returns: textContent (plain text), title, byline, excerpt
 |
 +-- textContent.length > 100? --> return (quality: medium)
 |
 v
html-to-text (last resort) -- convert(html, { skip: nav/footer/script })
 |  Converts ALL remaining HTML to plain text (not content-aware)
 |
 +-- text.length > 50? --> return (quality: low)
 |
 v
metadata-only -- Open Graph / meta tags only (quality: 0)
```

---

## Files to Create/Modify

### NEW: `src/extract.js` (~200 lines)

Core extraction module with these exports:

```javascript
import { Defuddle } from 'defuddle/node';
import { Readability, isProbablyReaderable } from '@mozilla/readability';
import { JSDOM } from 'jsdom';
import { convert as htmlToText } from 'html-to-text';

// Primary API: fetch URL and extract clean content
export async function fetchAndExtract(url, options = {}) { ... }

// Extract readable content from raw HTML string
export function extractContent(html, url) { ... }

// Extract Open Graph / meta tag metadata from HTML
export function extractMetadata(html) { ... }

// Score extraction quality (0-1)
export function scoreExtraction(result) { ... }
```

**`extractContent()` pipeline:**

1. Always extract metadata from HTML (OG tags, meta tags, schema.org via Defuddle)
2. Try Defuddle: `await Defuddle(html, url, { markdown: true })` -- if `content.length > 100`, return it
3. Try Readability: create JSDOM, check `isProbablyReaderable()`, parse -- if `textContent.length > 100`, return it
4. Try html-to-text: `convert(html, { selectors to skip nav/footer/script })` -- if length > 50, return it
5. Return metadata-only result with quality: 0

**`fetchAndExtract()` wraps the pipeline:**

1. HTTP GET with 30s timeout, User-Agent header
2. Check Content-Type header -- return descriptive note for PDFs and non-HTML
3. Read response text, cap at 500KB
4. Call `extractContent(html, url)`
5. Run paywall detection on raw HTML
6. Return unified result object

**Result object shape:**
```javascript
{
  content: string,      // Clean markdown or plain text
  title: string | null,
  author: string | null,
  date: string | null,   // From Defuddle's `published` field
  excerpt: string | null,
  siteName: string | null,
  wordCount: number,
  source: 'defuddle' | 'readability' | 'html-to-text' | 'metadata-only',
  quality: number,       // 0-1 score
  metadata: {            // Always present, from OG/meta/schema.org
    ogTitle, ogDescription, ogImage, siteName, title, description, author, ...
  },
  paywalled: boolean,
  url: string,
}
```

**Memory management:**
- Defuddle handles jsdom internally for its path
- For Readability fallback, create JSDOM and call `dom.window.close()` after parsing
- Sequential processing means only one DOM in memory at a time

### MODIFY: `src/processor.js`

**Change 1: Add import** (top of file)
```javascript
import { fetchAndExtract } from './extract.js';
```

**Change 2: Replace `fetchArticleContent()` body** (lines 619-648)

Replace the raw `fetch()` + 50KB truncation with a call to `fetchAndExtract()`. Return backward-compatible shape:
```javascript
export async function fetchArticleContent(url) {
  const result = await fetchAndExtract(url);
  return {
    text: result.content || '',
    title: result.title || null,
    author: result.author || null,
    excerpt: result.excerpt || null,
    wordCount: result.wordCount || 0,
    source: result.source || 'direct',
    paywalled: result.paywalled || false,
    metadata: result.metadata || {},
    quality: result.quality || 0,
  };
}
```

**Change 3: Update content storage** (lines 946-951 in `fetchAndPrepareBookmarks`)

Store extracted text with new metadata fields. Truncate clean text to 8KB (was 10KB of raw HTML; 8KB of clean text has 3-5x more useful info):
```javascript
content = {
  text: (fetchResult.text || '').slice(0, 8000) + (fetchResult.text?.length > 8000 ? '\n\n...[truncated]' : ''),
  title: fetchResult.title,
  author: fetchResult.author,
  excerpt: fetchResult.excerpt,
  source: fetchResult.source,
  paywalled: fetchResult.paywalled,
  quality: fetchResult.quality,
};
```

Add quality logging:
```javascript
console.log(`  Content: ${contentText.length} chars (${qualityLabel} via ${fetchResult.source})`);
```

**Change 4: Increase GitHub README limit** (line 595-596)

From 5000 to 8000 chars. READMEs are already markdown (no HTML bloat).

**Change 5: Remove thread tweet text truncation** (lines 1053-1055)

Remove the 500-char truncation on `threadTweets[].text`. Tweets are already length-limited by Twitter (max 4000 chars for Blue users). Full text needed for AI categorization.

### MODIFY: `src/index.js`

Add exports for the new extraction module.

### MODIFY: `package.json`

Add the four new dependencies.

### NEW: `test/extract.test.js`

Tests using Node.js built-in `node:test` and `node:assert` (matching project conventions):

- `extractContent()` with simple article HTML -- verifies nav/footer stripped, article text kept
- `extractContent()` with empty/script-only HTML -- returns metadata-only
- `extractContent()` with malformed HTML -- doesn't throw
- `extractMetadata()` with OG tags -- extracts all fields
- `extractMetadata()` with standard meta tags -- extracts title/description/author
- `extractMetadata()` with no metadata -- returns empty object
- `scoreExtraction()` quality ranges for different scenarios
- `fetchAndExtract()` timeout handling

Test fixtures in `test/fixtures/`:
- `article-simple.html` -- nav + article + footer
- `article-no-content.html` -- metadata only, no article body

### MODIFY: `test/processor.test.js`

Add tests for updated `fetchArticleContent()` return shape (new fields: title, author, quality, metadata).

---

## Storage Strategy

**Keep content in pending-bookmarks.json** (no separate files). Rationale:

- The real problem is raw HTML, not content volume. Clean extracted text is 5-10x smaller than raw HTML for the same article.
- 20 bookmarks with 17 article links: ~50-70KB clean text vs ~144KB raw HTML currently
- Stays well under the 256KB practical limit for Claude to read
- No changes needed to the AI processing command (process-bookmarks.md)
- The `content.source` field distinguishes old data ('direct') from new ('defuddle', 'readability', etc.)

Truncation limits after change:

| Content type | Current | New | Rationale |
|---|---|---|---|
| Article (extracted) | 10KB raw HTML | 8KB clean text | 3-5x more useful info per byte |
| GitHub README | 5KB | 8KB | Already markdown, matches article limit |
| Thread tweet text | 500 chars | No limit | AI needs full context |
| HTML fetch size | 50KB | 500KB | Extractors need full page structure |

---

## What Does NOT Change

- `src/cli.js` -- no new CLI flags needed
- `src/config.js` -- no new config options for v1
- `src/job.js` -- reads same JSON structure
- `.claude/commands/process-bookmarks.md` -- AI naturally benefits from cleaner content
- `fetchGitHubContent()` -- already clean (GitHub API returns structured data)
- `fetchXArticleContent()` -- already clean (bird CLI extracts content)
- `fetchContent()` router -- still dispatches GitHub/paywall/article correctly

---

## Implementation Order

1. `npm install defuddle jsdom @mozilla/readability html-to-text`
2. Create `src/extract.js` with extraction pipeline
3. Create `test/extract.test.js` with HTML fixtures and unit tests
4. Run `npm test` to validate extraction works in isolation
5. Update `src/processor.js` (replace fetchArticleContent, update storage, increase limits)
6. Update `src/index.js` with new exports
7. Run full test suite
8. Manual integration test: `npx smaug fetch 5` and inspect pending-bookmarks.json

---

## Verification

1. **Unit tests pass:** `npm test`
2. **Manual comparison:** Fetch same bookmarks before/after, compare JSON content quality
3. **File size check:** pending-bookmarks.json should be similar or smaller than before
4. **Quality logging:** Console output shows extraction quality per article
5. **Fallback test:** Verify Readability fallback activates when Defuddle returns empty content
6. **Error resilience:** Malformed HTML, timeouts, and non-HTML URLs handled gracefully

---

## Libraries Evaluated But Not Selected

| Library | Verdict | Reason |
|---------|---------|--------|
| **Playwright** | Not recommended | 500MB+ install (browser binaries), 100-300MB RAM per page, massive overkill for article extraction from mostly server-rendered pages |
| **Puppeteer** | Not recommended | Same category as Playwright -- Chromium-only, heavy footprint |
| **Cheerio** | Not recommended | Low-level HTML parsing; requires writing custom extraction heuristics per site layout. Defuddle/Readability already solve this |
| **Jina Reader** (`r.jina.ai`) | Deferred | External API dependency, ~8s latency per URL, rate limited (20 RPM free). Good for JS-rendered pages but adds fragility. Consider as future opt-in fallback |
| **Scrapy / Scrapling** | Not applicable | Python-only frameworks, project is Node.js |

---

## Future Enhancements (not part of this change)

- **Content caching** -- cache extracted content by URL hash in `.state/content-cache/`
- **PDF extraction** -- use `pdf-parse` for PDF URLs (currently returns "not supported" note)
- **Jina Reader fallback** -- proxy through `r.jina.ai` for JS-rendered pages when local extraction fails (requires opt-in config)
- **Smart paywall detection** -- use extraction quality as paywall signal (good domain + low quality = likely paywalled)
- **Extraction statistics** -- aggregate quality metrics across runs
