# Plan: `.md` Filename Filter Cleanup (v2)

**Date:** 2026-02-08
**Scope:** `src/processor.js` and `test/processor.test.js`
**Supersedes:** `docs/plan-md-filename-filter-cleanup.md`

---

## Key Architectural Change

The original implementation added `isLikelyMdFilename()` as a **pre-filter** that ran before the type classification chain (line 991), used `continue` to bypass it, and duplicated the link-tracking logic. This was unnecessary — the existing type chain already handles types that don't need content fetching (e.g., `video`, `image`, `media` all get `content = null` naturally).

**The fix:** Move `.md` detection *into* the type classification chain as another `else if` branch, following the same pattern as every other link type. This eliminates the separate integration point, the `continue`, the duplicate tracking code, and `_filenameDetection`.

This reframing cascades into a much deeper simplification than the v1 plan anticipated.

---

## Why `KNOWN_DEV_FILENAMES` is no longer needed

The function had three layers that all returned `isFilename: true` — the only difference was the `confidence` level (`high`, `medium`, `low`). Since nothing downstream ever reads `confidence`, all three layers produce the same outcome. With confidence removed:

- Layer 1 (known filenames → high) → `return true`
- Layer 2 (tweet context → medium) → `return true` (also being removed)
- Layer 3 (bare domain catch-all → low) → `return true`

They all collapse to a single `return true`. The 40-entry `KNOWN_DEV_FILENAMES` Set was only useful for assigning higher confidence to recognized names — not for changing the boolean result. Without confidence, the Set has no effect on behavior.

The function's logic simplifies to: **"Is this a bare .md root domain that isn't on the allowlist?"**

---

## Planned Changes

### `src/processor.js`

#### 1. Delete constants that are no longer needed

- `LEGITIMATE_MD_DOMAINS` array (line 221-226) — replaced by inline check
- `KNOWN_DEV_FILENAMES` Set (line 232-245) — no longer changes behavior
- `FILE_CONTEXT_PATTERNS` array (line 248-262) — Layer 2 removed entirely

#### 2. Rewrite `isLikelyMdFilename` (lines 264-340 → ~15 lines)

**Before:** Returns `{ isFilename: boolean, confidence: string, reason: string }`, accepts `tweetText` parameter, has 3 detection layers.

**After:** Returns `boolean`, no `tweetText` parameter, single pass:

```javascript
export function isLikelyMdFilename(expandedUrl) {
  let url;
  try {
    url = new URL(expandedUrl);
  } catch {
    return false;
  }

  if (!url.hostname.endsWith('.md')) return false;

  // Allowlist: obsidian.md is the only known legitimate .md domain in tech Twitter
  if (url.hostname === 'obsidian.md' || url.hostname.endsWith('.obsidian.md')) return false;

  // Subdomained .md domains (e.g., www.plan.md) are more likely real websites
  if (url.hostname.split('.').length > 2) return false;

  // URLs with a meaningful path are likely intentional links to real pages
  if (url.pathname.replace(/\/$/, '').length > 0) return false;

  // Bare .md root domain — overwhelmingly an auto-linked filename in tech Twitter
  // TODO: ALL_CAPS detection needs bird CLI's entities.urls[].display_url
  return true;
}
```

Behavior is identical — every input that returned `isFilename: true` now returns `true`, and vice versa.

#### 3. Move integration into the type chain

**Delete** the pre-filter block (lines 991-1008):
```javascript
// DELETE THIS ENTIRE BLOCK:
const mdCheck = isLikelyMdFilename(expanded, text);
if (mdCheck.isFilename) {
  console.log(`  Skipping .md filename auto-link: ${expanded} (${mdCheck.reason})`);
  const linkData = {
    original: link,
    expanded,
    type: 'filename-reference',
    content: null,
    _filenameDetection: mdCheck
  };
  allLinks.push(linkData);
  const sourceTweets = tweetLinkSources.get(link) || [];
  if (sourceTweets.includes(bookmark.id)) {
    links.push(linkData);
  }
  continue;
}
```

**Add** an `else if` branch in the type classification chain (after `github.com`, before `youtube.com`):
```javascript
} else if (isLikelyMdFilename(expanded)) {
  type = 'filename-reference';
  console.log(`  Skipping .md filename auto-link: ${expanded}`);
}
```

This means:
- `content` stays `null` (initialized on line 1012)
- `linkData` is built by the shared code at line 1100-1105
- Link tracking uses the shared code at lines 1107-1114
- No `_filenameDetection` debug data in production JSON
- No duplicate push logic

### `test/processor.test.js`

#### Keep (adapted to boolean assertions)

| Test | Asserts |
|------|---------|
| Known filenames — parameterized sample | `true` for representative set in a loop |
| Obsidian.md allowlist | `false` |
| Obsidian.md subdomain (help.obsidian.md) | `false` |
| Non-.md domains — parameterized | `false` for github.com, example.com, x.com |
| Invalid URL | `false` |
| Subdomained .md (www.plan.md) | `false` |
| Subdomained .md (app.something.md) | `false` |
| .md domain with path | `false` |
| .md domain with trailing slash only | `true` |
| .md domain without trailing slash | `true` |
| Unknown bare .md domain | `true` |

#### Add

| Test | Asserts |
|------|---------|
| Query params on root (`plan.md/?utm=twitter`) | `true` |
| Hash fragment on root (`plan.md/#section`) | `true` |
| Path with query params (`plan.md/pricing?ref=123`) | `false` |
| http:// works same as https:// | `true` |

#### Remove

| Test(s) | Reason |
|---------|--------|
| 11 individual known filename tests | Parameterized into 1 |
| URL normalization section (2 tests) | Duplicate of known filenames (URL spec lowercases before function runs) |
| Layer 2 context tests (5 tests) | Feature removed |
| "Real-world cases" (2 tests) | Just retested Layer 1 |
| Return structure validation (1 test) | No more object return type |
| All `confidence` assertions | Field removed |

#### Estimated result: ~14 tests covering more scenarios than the original 33

---

## Summary

| Metric | Before | v1 Plan | v2 Plan (this) |
|--------|--------|---------|----------------|
| `processor.js` lines added | ~120 | ~45 | ~15 |
| Constants/data structures | 3 (Set, array, array) | 1 (Set) | 0 |
| Function return type | `{ isFilename, confidence, reason }` | `{ isFilename, reason }` | `boolean` |
| Parameters | `(expandedUrl, tweetText)` | `(expandedUrl)` | `(expandedUrl)` |
| Integration pattern | Pre-filter with `continue` | Pre-filter with `continue` | Branch in type chain |
| Duplicate tracking code | Yes | Yes (accepted) | No |
| `_filenameDetection` in JSON | Yes | No | No |
| Test count | 33 | ~16 | ~14 |
| Edge cases covered | 0 | 4+ | 4+ |
| Regex patterns to maintain | 13 | 0 | 0 |

---

## Not changing

- **Subdomain escape hatch** — `www.plan.md` returns false. Pragmatically correct (Twitter's auto-linker doesn't generate subdomained URLs)
- **Function name** — `isLikelyMdFilename` is clear in context
- **Allowlist approach** — obsidian.md checked inline rather than in an array (only one entry)
- **Function stays exported** — needed for unit tests
