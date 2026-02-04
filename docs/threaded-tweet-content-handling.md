# Threaded Tweet Content Handling

This document explains how Smaug handles Twitter/X threads when processing bookmarks, including the data flow, aggregation logic, and JSON structure.

## Overview

When you bookmark the first tweet of a thread, Smaug's **thread expansion** feature automatically fetches the author's self-reply chain and captures content from ALL tweets in the thread. This solves a common problem where important links (GitHub repos, articles) appear in tweet #2 or later.

**Key insight:** Links are aggregated into a single array, but tweet texts are stored separately (not concatenated).

## Architecture: Three-Phase Pipeline

```
┌──────────────────────────────────────────────────────────────────┐
│ PHASE 1: FETCH (bird CLI)                                        │
│                                                                  │
│ npx smaug fetch  →  bird bookmarks --author-chain --thread-meta  │
│                     ↓                                            │
│              Returns flat array of tweets with thread metadata   │
│              (isThread, threadPosition, threadRootId, etc.)      │
└──────────────────────────────────────────────────────────────────┘
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│ PHASE 2: AGGREGATION (processor.js - JavaScript)                 │
│                                                                  │
│ groupThreadTweets() at line 277:                                 │
│   - Groups tweets by conversationId/threadRootId                 │
│   - Returns: { primaryTweet, threadTweets[], threadId, ... }     │
│                                                                  │
│ fetchAndPrepareBookmarks() at lines 809-972:                     │
│   - Iterates ALL threadTweets to collect t.co links              │
│   - Expands each link (t.co → real URL)                          │
│   - Tracks which tweet each link came from (tweetLinkSources)    │
│   - Builds allLinks[] from ALL thread tweets                     │
│   - Builds threadTweetsData[] with per-tweet link arrays         │
└──────────────────────────────────────────────────────────────────┘
                              ↓
                 .state/pending-bookmarks.json
                 (already aggregated)
                              ↓
┌──────────────────────────────────────────────────────────────────┐
│ PHASE 3: AI PROCESSING (Claude + process-bookmarks.md)           │
│                                                                  │
│ Claude reads pre-aggregated JSON and follows markdown rules:     │
│   "When isThread: true, use allLinks[] for categorization"       │
│                                                                  │
│ The AI doesn't aggregate - it interprets aggregated data         │
└──────────────────────────────────────────────────────────────────┘
```

## What "Aggregation" Means

### Links: Truly Aggregated

Links from all tweets in a thread are collected into a single `allLinks[]` array. This is the primary aggregation that happens in processor.js.

```javascript
// processor.js:809-828
const allTcoLinks = new Set();
const tweetLinkSources = new Map(); // link -> tweet IDs that contain it

for (const tweet of threadTweets) {
  const tweetText = tweet.text || tweet.full_text || '';
  const linkMatches = tweetText.match(/https?:\/\/t\.co\/\w+/g) || [];

  for (const link of linkMatches) {
    allTcoLinks.add(link);
    // Track which tweet(s) contain this link
    if (!tweetLinkSources.has(link)) {
      tweetLinkSources.set(link, []);
    }
    tweetLinkSources.get(link).push(tweet.id);
  }
}
```

**Note on `tweetLinkSources`:** This Map tracks which tweet(s) contain each link. This is useful for:
- Debugging: "Where did this link come from?"
- Attribution: Knowing if a link appeared in multiple tweets
- Future enhancement: Could be exposed in JSON for richer context

### Text: NOT Concatenated

Tweet texts are stored **separately** in the `threadTweets[]` array, not concatenated into a single field. Each tweet's text is preserved individually, though truncated to 500 characters to prevent JSON bloat.

```javascript
// processor.js:1052-1055
const tweetText = t.text || t.full_text || '';
const truncatedText = tweetText.length > 500
  ? tweetText.slice(0, 500) + '...'
  : tweetText;
```

## JSON Structure for Threaded Bookmarks

### Field Summary

| Field | Scope | Aggregated? | Purpose |
|-------|-------|-------------|---------|
| `text` | Primary tweet only | N/A | Full text, backward compatibility |
| `links[]` | Primary tweet only | N/A | Backward compatibility |
| `isThread` | Boolean | N/A | Indicates thread expansion occurred |
| `threadPosition` | String | N/A | "root", "middle", "end", or "standalone" |
| `threadRootId` | String | N/A | ID of thread's first tweet |
| `threadTweets[]` | All tweets | No (separate objects) | Per-tweet data with truncated text |
| `threadTweets[].text` | Per-tweet | No | Truncated to 500 chars |
| `threadTweets[].links[]` | Per-tweet | No | Links from that specific tweet |
| `allLinks[]` | All thread tweets | **Yes** (flat array) | For AI categorization |

### Example: 3-Tweet Thread

```json
{
  "id": "2017982342854218005",
  "author": "alex_prompter",
  "text": "Steal my OpenClaw system prompt...[FULL TEXT ~4000 chars]",

  "links": [
    {
      "original": "https://t.co/YSz85YYwut",
      "expanded": "https://memory.md/",
      "type": "article",
      "content": {...}
    }
  ],

  "isThread": true,
  "threadPosition": "root",
  "threadRootId": "2017982342854218005",

  "threadTweets": [
    {
      "id": "2017982342854218005",
      "text": "Steal my OpenClaw system prompt...[TRUNCATED TO 500 chars]...",
      "createdAt": "Sun Feb 01 15:24:17 +0000 2026",
      "threadPosition": "root",
      "links": [
        {"original": "https://t.co/YSz85YYwut", "expanded": "https://memory.md/", "type": "article"}
      ]
    },
    {
      "id": "2017982542935134650",
      "text": "How to use:\n1/ Save as https://t.co/iETv5iJpM5 in your OpenClaw root...",
      "createdAt": "Sun Feb 01 15:25:05 +0000 2026",
      "threadPosition": "middle",
      "links": [
        {"original": "https://t.co/iETv5iJpM5", "expanded": "https://code.claude.com/docs", "type": "article"}
      ]
    },
    {
      "id": "2017982626829574553",
      "text": "Your premium AI bundle to 10x your business...",
      "createdAt": "Sun Feb 01 15:25:25 +0000 2026",
      "threadPosition": "end",
      "links": [
        {"original": "https://t.co/ZKcpVsalBb", "expanded": "https://www.godofprompt.ai/complete-ai-bundle", "type": "article"}
      ]
    }
  ],

  "allLinks": [
    {"original": "https://t.co/YSz85YYwut", "expanded": "https://memory.md/", "type": "article", "content": {...}},
    {"original": "https://t.co/iETv5iJpM5", "expanded": "https://code.claude.com/docs", "type": "article", "content": {...}},
    {"original": "https://t.co/ZKcpVsalBb", "expanded": "https://www.godofprompt.ai/complete-ai-bundle", "type": "article", "content": {...}}
  ]
}
```

### Comparison: Threaded vs Non-Threaded

| Field | Non-Thread | Threaded |
|-------|------------|----------|
| `links[]` | All links from tweet | Only primary tweet links |
| `allLinks[]` | Not present | Aggregated from ALL thread tweets |
| `isThread` | `false` | `true` |
| `threadTweets[]` | Not present | Array with per-tweet text + links |
| `threadPosition` | `"standalone"` | `"root"`, `"middle"`, or `"end"` |
| `threadRootId` | `null` | ID of first tweet in thread |

## Key Code Locations

| Function | File | Line | Purpose |
|----------|------|------|---------|
| `groupThreadTweets()` | src/processor.js | 277-324 | Groups tweets by conversationId/threadRootId |
| `fetchAndPrepareBookmarks()` | src/processor.js | 685-1155 | Main processing, builds allLinks[] |
| Link collection loop | src/processor.js | 809-857 | Iterates threadTweets for t.co links |
| Text truncation | src/processor.js | 1052-1055 | Caps threadTweets[].text at 500 chars |
| threadTweetsData building | src/processor.js | 1049-1081 | Creates per-tweet data objects |

## Controller Responsibility: Node.js vs Claude

A key architectural insight is understanding **who controls what** in the processing pipeline:

| Responsibility | Controller | Location | Notes |
|----------------|------------|----------|-------|
| Fetching tweets | Node.js | processor.js | Calls bird CLI |
| Grouping threads | Node.js | processor.js:277 | `groupThreadTweets()` |
| Link aggregation | Node.js | processor.js:809-857 | Builds `allLinks[]` |
| Text storage | Node.js | processor.js:1049-1081 | Builds `threadTweets[]` |
| Link expansion (t.co) | Node.js | processor.js | HTTP HEAD requests |
| Content fetching | Node.js | processor.js | GitHub API, articles |
| **Categorization** | **Claude** | process-bookmarks.md:187 | Uses `allLinks[]` |
| **Title generation** | **Claude** | process-bookmarks.md:281 | Reads all `threadTweets[].text` |
| **Filing decisions** | **Claude** | process-bookmarks.md | knowledge/ vs bookmarks.md |
| **Markdown formatting** | **Claude** | process-bookmarks.md | Entry presentation |

**Key insight:** Claude is an **interpreter** of pre-aggregated data, not an aggregator. All the grouping, link collection, and content fetching happens in Phase 1/2 (Node.js). Claude receives ready-to-use data and makes decisions about categorization and presentation.

## Backward Compatibility Design

The JSON structure maintains **two sets of link fields** for backward compatibility:

| Field | Scope | Purpose |
|-------|-------|---------|
| `links[]` | Primary tweet only | Backward compatibility with non-thread code paths |
| `allLinks[]` | All thread tweets | Thread-aware processing (use this for categorization!) |

This design means:
- Old code that only reads `links[]` still works (gets primary tweet links)
- New thread-aware code uses `allLinks[]` for complete coverage
- Non-threaded bookmarks don't have `allLinks[]` (unnecessary duplication)

## How AI Processing Uses Thread Data

The instructions in `.claude/commands/process-bookmarks.md` tell Claude:

1. **Use `allLinks[]` for categorization** when `isThread: true` - the important link (GitHub repo, etc.) may be in a later tweet
2. **Read all `threadTweets[].text`** to understand full thread context when creating titles/descriptions
3. **Format thread entries** with numbered tweet summaries in the blockquote

Example markdown output format:
```markdown
## @author - Title Based on Full Thread Content
> **Thread (3 tweets):**
> 1. First tweet text truncated to 100 chars...
> 2. Second tweet text truncated to 100 chars...
> 3. Third tweet text truncated to 100 chars...

- **Thread:** https://x.com/author/status/123 (links to root tweet)
- **Link:** https://github.com/... (from allLinks[])
- **Filed:** [repo-name](./knowledge/tools/repo-name.md)
- **What:** Description based on full thread context
```

## Configuration Options

### Config File (smaug.config.json)

```json
{
  "expandThreads": true,                    // Enable/disable thread expansion (default: true)
  "threadExpansionMode": "author-chain"     // "author-chain", "author-only", or "none"
}
```

### CLI Flag

```bash
npx smaug fetch --no-threads   # Disable thread expansion for this fetch
```

### Thread Expansion Modes

| Mode | Description |
|------|-------------|
| `author-chain` | Connected self-reply chain only (recommended, default) |
| `author-only` | All author tweets in thread (even disconnected replies) |
| `none` | Disable thread expansion |

## Performance Notes

- Thread expansion adds ~1 second per bookmark (bird CLI rate limiting)
- Timeout is increased by 50% when thread expansion is enabled
- Text truncation at 500 chars prevents JSON bloat for long threads

---

## Critical Issue: Content Truncation Throughout Codebase

**Investigation Date:** February 4, 2026

A comprehensive audit revealed that content truncation happens at **multiple points** in the processing pipeline, and predates the thread expansion feature. More critically, **article content is stored as raw HTML, not readable text**.

### All Truncation Points in `src/processor.js`

| Location | Target | Limit | When Added | Severity |
|----------|--------|-------|------------|----------|
| Line 596 | GitHub README | 5,000 chars | **PRE-EXISTING** | Medium |
| Line 635 | Article raw fetch | 50,000 chars | **PRE-EXISTING** | High |
| Line 947 | Article stored in JSON | 10,000 chars | **PRE-EXISTING** | **CRITICAL** |
| Line 1055 | Thread tweet text | 500 chars | **NEW (thread expansion)** | Medium |

### The Double-Truncation Problem

Article content is truncated **TWICE**, losing 80% of content:

```
fetchArticleContent()          fetchAndPrepareBookmarks()
        ↓                              ↓
  .slice(0, 50000)              .slice(0, 10000)
   (50KB limit)                  (10KB limit)
        ↓                              ↓
   First truncation ──────────► Second truncation
                                       ↓
                              Only 10KB stored in JSON
```

**Code locations:**
```javascript
// Line 635 - First truncation
const result = text.slice(0, 50000);

// Line 947 - Second truncation
content = {
  text: fetchResult.text?.slice(0, 10000),  // 80% loss!
  source: fetchResult.source,
  paywalled: fetchResult.paywalled
};
```

### Critical: Raw HTML Storage (No Text Extraction)

**The stored content is raw HTML, not readable text.**

Example from actual `pending-bookmarks.json`:
```json
"content": {
  "text": "<!DOCTYPE html><html lang=\"en\" class=\"__variable_ed9089...<head><meta charset=\"utf-8\"/>..."
}
```

**Problems with this approach:**
1. **Wasted storage** - HTML tags are ~70% of bytes
2. **Noisy for AI** - Claude must parse through `<script>`, `<style>`, navigation
3. **Wrong truncation point** - Often cuts mid-tag or mid-word
4. **Missing actual content** - 10KB of HTML often captures only boilerplate headers

**Root cause:** No HTML-to-text conversion library is installed. The `fetchArticleContent()` function (lines 619-648) does a simple `response.text()` and returns raw HTML.

```javascript
// Current implementation - NO parsing
const response = await fetch(url, {...});
const text = await response.text();  // Raw HTML!
const result = text.slice(0, 50000);
return { text: result, source: 'direct', paywalled: false };
```

### What SHOULD Happen

```
URL → fetch() → raw HTML
                   ↓
         HTML-to-text extraction (e.g., @mozilla/readability)
                   ↓
         Clean article text (~10-20% of original size)
                   ↓
         Store full readable content (no truncation needed)
```

### GitHub README: Better But Still Truncated

GitHub content is handled better - it's already markdown from the API:

```javascript
// Line 594-597
readme = Buffer.from(readmeJson.content, 'base64').toString('utf8');
if (readme.length > 5000) {
  readme = readme.slice(0, 5000) + '\n...[truncated]';
}
```

At least it's readable text, but 5KB may cut off important installation instructions or API documentation.

### Thread Tweet Text: New Truncation

Added with thread expansion feature:

```javascript
// Lines 1053-1055
const truncatedText = tweetText.length > 500
  ? tweetText.slice(0, 500) + '...'
  : tweetText;
```

**Impact:** Long-form tweets (up to 4000 chars for Twitter Blue users) lose significant content. The primary tweet's `text` field is NOT truncated, but `threadTweets[].text` entries are.

### Example: Real Thread Analysis

Tweet ID `2018323558746014087` (@orenyomtov) - 11-tweet security research thread:

| Tweet # | Text Length | Truncated? | Links |
|---------|-------------|------------|-------|
| 1 | 220 chars | No | 1 (photo) |
| 2 | 274 chars | No | 0 |
| 3 | 243 chars | No | 0 |
| 4 | 262 chars | No | 1 |
| 5 | 308 chars | No | 0 |
| 6 | 254 chars | No | 0 |
| 7 | 277 chars | No | 0 |
| 8 | 247 chars | No | 0 |
| 9 | 242 chars | No | 0 |
| 10 | 113 chars | No | 1 |
| 11 | 93 chars | No | 1 |

This thread was lucky - all tweets under 500 chars. But threads with long-form content WILL be truncated.

---

## Tasks: Changes Needed / Improvements to Prioritize

### CRITICAL Priority (Content Loss)

- [ ] **Remove thread tweet text truncation** - Currently truncates to 500 chars, which loses content from long-form tweets. Twitter Blue allows 4000-char tweets. **Decision: Remove truncation entirely** - tweets are already length-limited by Twitter, no need to truncate further. (Line 1055 in processor.js)

- [ ] **Redesign article/webpage content extraction (NEEDS DISCUSSION)** - Current approach downloads raw HTML and truncates it twice. Proposed new approach:
  1. Download **full HTML** with no truncation during fetch
  2. Use a parsing library to extract **only human-readable content**
  3. Store the clean extracted text (which will be much smaller than raw HTML)

  **Topics requiring discussion before implementation:**
  - **Library comparison:** Review parsing solutions:
    - `@mozilla/readability` - Mozilla's reader mode (best quality, used by Firefox)
    - `html-to-text` - Simple conversion, less intelligent
    - `r.jina.ai` API - External service, no local processing
    - `cheerio` + custom extraction - Maximum control, most work
  - **Processing strategy:** Download full HTML then parse, or stream/parse in realtime?
  - **Memory concerns:** Holding full HTML pages in memory while processing many bookmarks - is this a concern? Current implementation processes bookmarks sequentially, but parallel processing could increase memory pressure.
  - **Error handling:** What if parsing fails? Fall back to raw HTML? Skip content entirely?
  - **Storage:** Store only extracted text, or keep raw HTML as backup?

### High Priority

- [ ] **Increase GitHub README limit from 5KB to 10KB** - READMEs are already content-dense markdown (no HTML bloat), so truncation is acceptable but current limit is too low. Double it to capture more installation instructions and API docs. (Line 596 in processor.js)

- [ ] **Add thread text summary field** - Concatenate all `threadTweets[].text` into a single `threadSummary` field for easier AI consumption. Currently the AI must read separate array entries.

### Medium Priority

- [ ] **Add performance note to CLI help** - Document that thread expansion adds ~1 second per bookmark in the `fetch` command help output.

- [ ] **Integration test for full pipeline** - Mock bird CLI and test the complete flow from fetch through to pending-bookmarks.json structure (deferred from Feb 3 session due to complexity).

- [ ] **Audit X article content extraction** - The `fetchXArticleContent()` function may have similar issues. Verify it extracts actual article text, not just metadata.

### Low Priority / Future Enhancements

- [ ] **Thread summary generation** - Use AI to create a coherent summary of the entire thread rather than just listing tweets.

- [ ] **Configurable README truncation** - Add `readmeMaxLength` config option (default 10KB) for users who want more/less README content.

- [ ] **Thread metrics** - Track statistics like "links discovered from non-primary tweets" to demonstrate thread expansion value.

- [ ] **Content extraction quality metrics** - Log warnings when parsing fails or content seems incomplete.

### Completed (Feb 2-3, 2026)

- [x] Implement `groupThreadTweets()` function
- [x] Build `allLinks[]` aggregation in `fetchAndPrepareBookmarks()`
- [x] Add `threadTweets[]` with per-tweet link tracking
- [x] Add `--no-threads` CLI flag
- [x] Update AI processing instructions for thread handling
- [x] Add 13 unit tests for `groupThreadTweets()`
- [x] Add 4 edge case tests (empty threads, standalone tweets, link aggregation, threadPosition preservation)
- [x] E2E validation: confirmed links from tweet #2 captured correctly

### Investigated (Feb 4, 2026)

- [x] Audit all truncation points in codebase
- [x] Identify raw HTML storage issue (no text extraction)
- [x] Document double-truncation problem
- [x] Analyze real thread examples for truncation impact

---

## Related Resources

- **Interactive Visualization:** `smaug-dataflow.html` - Click "Thread" tweet type and enable "Thread Expansion" toggle to see the data flow animated
- **Visualization Spec:** `docs/smaug-dataflow-spec.md` - Technical details about the playground
- **AI Instructions:** `.claude/commands/process-bookmarks.md` - What Claude is told to do with thread data
- **Source Code:** `src/processor.js` - Implementation of grouping, aggregation, and JSON building
