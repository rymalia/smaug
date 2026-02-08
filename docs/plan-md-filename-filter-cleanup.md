# Plan: `.md` Filename Filter Cleanup

**Date:** 2026-02-08
**Scope:** `src/processor.js` and `test/processor.test.js`
**Commit under review:** `a24d741`

---

## Background

The `.md` filename filter (`isLikelyMdFilename`) detects when Twitter auto-links bare filenames like `CLAUDE.md` as t.co redirects to Moldova's `.md` ccTLD. It works correctly — 62 links scanned in real data, 3 false positives caught, 0 false negatives. But a design review revealed the implementation is over-engineered for the problem it solves.

---

## Issues Found

### Design Issues (Architectural)

#### 1. `LEGITIMATE_MD_DOMAINS` — redundant entries
**File:** `processor.js:221-226`

The allowlist has 4 entries but only needs 1. The `endsWith('.' + d)` check already catches all subdomains of `obsidian.md`:

```javascript
// Current (redundant)
const LEGITIMATE_MD_DOMAINS = [
  'obsidian.md',
  'help.obsidian.md',   // already caught by endsWith('.obsidian.md')
  'forum.obsidian.md',  // already caught by endsWith('.obsidian.md')
  'publish.obsidian.md', // already caught by endsWith('.obsidian.md')
];

// Should be
const LEGITIMATE_MD_DOMAINS = ['obsidian.md'];
```

#### 2. `confidence` field has inconsistent semantics and is never used
**File:** `processor.js:277-340`, integration at `processor.js:992-993`

The return type includes `confidence: 'none' | 'low' | 'medium' | 'high'` but:
- Its meaning flips depending on `isFilename` (e.g., `isFilename: false, confidence: 'high'` means "confident it's NOT a filename"; `isFilename: true, confidence: 'high'` means "confident it IS")
- The only call site never reads it:
  ```javascript
  const mdCheck = isLikelyMdFilename(expanded, text);
  if (mdCheck.isFilename) {  // <-- confidence ignored
  ```
- It's stored in `_filenameDetection` in the JSON output but nothing downstream uses it

**Fix:** Drop `confidence` from the return type. Keep `reason` (useful for log messages).

#### 3. Layer 2 (tweet context analysis) is speculative engineering
**File:** `processor.js:247-262` (`FILE_CONTEXT_PATTERNS`), `processor.js:321-331`

From real data testing: 62 links scanned, 3 `.md` links found, **all caught by Layer 1** (known filenames). Layer 2 has never fired.

It adds 13 regex patterns to maintain — some overly broad (`/\bfiles?\b/` matches "I file my taxes", `/\bcursor\b/` matches "move your cursor") — for a case that hasn't been observed. The catch-all (Layer 3) would handle the same URLs anyway.

**Fix:** Remove `FILE_CONTEXT_PATTERNS` and Layer 2 entirely. Remove the `tweetText` parameter.

#### 4. `_filenameDetection` leaks debug data into production JSON
**File:** `processor.js:1000`

```javascript
const linkData = {
    original: link,
    expanded,
    type: 'filename-reference',
    content: null,
    _filenameDetection: mdCheck  // diagnostic object in pending-bookmarks.json
};
```

The underscore convention suggests "internal" but it's serialized into `pending-bookmarks.json` consumed by the AI categorizer. The categorizer only needs `type: 'filename-reference'`.

**Fix:** Remove `_filenameDetection` from link data.

### Logic Issues

#### 5. Tweet context uses the WRONG text for thread links
**File:** `processor.js:992`

```javascript
const mdCheck = isLikelyMdFilename(expanded, text);
```

`text` is the **primary tweet's** text (line 919). But the link might come from thread tweet #3. If thread tweet #3 mentions "scratchpad.md" and links to it, context analysis runs against tweet #1's text instead.

Harmless in practice (Layer 1 catches known filenames regardless, and the catch-all handles the rest), but logically wrong. Becomes moot when Layer 2 is removed.

#### 6. `FILE_CONTEXT_PATTERNS` has overly broad matches
**File:** `processor.js:248-262`

- `/\bfiles?\b/` — matches "I file my taxes"
- `/\bcursor\b/` — matches "move your cursor to the left"
- `/\bcache\s+.*\bin\b/` — greedy `.*` matches across entire sentences

Becomes moot when Layer 2 is removed.

#### 7. Subdomain escape hatch is debatable
**File:** `processor.js:295-298`

`www.claude.md/` escapes the filter while `claude.md/` doesn't. The assumption "subdomains = real website" is hand-wavy. Twitter's auto-linker wouldn't generate `www.plan.md` though, so this is theoretical. Keep as-is but acknowledge the reasoning is pragmatic, not rigorous.

### Code Quality

#### 8. Duplicated link-tracking pattern in integration
**File:** `processor.js:996-1007` vs `processor.js:1106-1113`

The filename branch duplicates the same `allLinks.push` + `sourceTweets.includes` pattern used for all other links. The `continue` forces duplication.

**Fix:** Accept the duplication — restructuring to share the tracking would add complexity for minimal gain. The pattern is only 5 lines.

#### 9. Three-line comment for a non-existent feature
**File:** `processor.js:317-319`

```javascript
// Note: ALL_CAPS detection (CLAUDE.md, MY_PROJECT.md) is not possible here because
// URL hostnames are normalized to lowercase by the URL spec. This could be implemented
// in the future if bird CLI exposes entities.urls[].display_url (the original text).
```

Design note masquerading as a code comment.

**Fix:** Replace with a single-line TODO: `// TODO: ALL_CAPS detection needs bird CLI's entities.urls[].display_url`

#### 10. Semantic naming: `isFilename` on obsidian.md
**File:** `processor.js:289-291`

`isLikelyMdFilename('https://obsidian.md/')` returns `{ isFilename: false }` — but `obsidian.md` literally IS a `.md` filename. The return value means "should we filter this" not "is this a filename." The function name is fine if you read it as "is this likely an *auto-linked* .md filename," but the field name `isFilename` is ambiguous.

**Fix:** Accept as-is. Renaming the function or field would be bike-shedding. The JSDoc and usage context make the intent clear enough.

### Test Issues

#### 11. 11 tests for `Set.has()` — massive repetition
**File:** `test/processor.test.js:928-993`

Tests for `plan.md`, `memory.md`, `claude.md`, `readme.md`, `todo.md`, `changelog.md`, `contributing.md`, `security.md`, `rules.md`, `agents.md` all test the same code path: `KNOWN_DEV_FILENAMES.has(baseName)`. After the first 2-3, you're just testing that JavaScript's `Set.has()` works.

**Fix:** Parameterize into one test with a representative sample.

#### 12. URL normalization tests are misleading
**File:** `test/processor.test.js:996-1012`

```javascript
test('ALL_CAPS URLs are lowercased by URL spec, fall through to known list or default', () => {
    const result = isLikelyMdFilename('https://CLAUDE.md/');
```

This tests the exact same thing as the `claude.md` test — `new URL()` lowercases the hostname before the function runs. The test name implies the function handles casing; it doesn't.

**Fix:** Keep one case-insensitivity test but rename it honestly: "URL spec lowercases hostnames."

#### 13. "Real-world cases" don't test real-world paths
**File:** `test/processor.test.js:1149-1163`

```javascript
test('plan.md from @threepointone tweet', () => {
    const result = isLikelyMdFilename('https://plan.md/', tweetText);
    assert.strictEqual(result.confidence, 'high'); // 'plan' is in known filenames
```

The comment says it all — `plan` is in the known list, so the tweet text is irrelevant. Layer 1 catches it before Layer 2 runs. These tests don't exercise the contextual paths they appear to test.

**Fix:** Remove. The known filename tests already cover this.

#### 14. Missing edge case tests

No tests for:
- Query parameters: `https://plan.md/?utm_source=twitter`
- Hash fragments: `https://plan.md/#section`
- Empty string input
- `http://` vs `https://`

**Fix:** Add these.

---

## Planned Changes

### `src/processor.js`

1. **Simplify `LEGITIMATE_MD_DOMAINS`** to `['obsidian.md']`
2. **Remove `FILE_CONTEXT_PATTERNS`** (all 13 regexes) and the Layer 2 block
3. **Remove `tweetText` parameter** from `isLikelyMdFilename`
4. **Drop `confidence` from return type** — return `{ isFilename: boolean, reason: string }`
5. **Replace 3-line ALL_CAPS comment** with 1-line TODO
6. **Remove `.toLowerCase()` on baseName** — URL spec already lowercases hostnames
7. **Remove `_filenameDetection`** from link data at integration point
8. **Update call site** to drop `text` argument: `isLikelyMdFilename(expanded)` instead of `isLikelyMdFilename(expanded, text)`
9. **Simplify JSDoc** to match new return type

**Estimated impact:** ~120 lines → ~45 lines (~60% reduction)

### `test/processor.test.js`

1. **Parameterize known filename tests** — 11 tests → 1 test with representative sample loop
2. **Remove URL normalization section** — fold the one useful test (case-insensitivity) into known filenames
3. **Remove Layer 2 context tests** — feature being removed
4. **Remove misleading "real-world cases"** — just retested Layer 1
5. **Add edge case tests:**
   - Query params on root path (`plan.md/?utm_source=twitter`) — should filter
   - Path with query params (`plan.md/pricing?ref=123`) — should NOT filter
   - Hash fragment on root path (`plan.md/#section`) — should filter
   - Empty string — should not crash
6. **Keep:** subdomain tests, path tests, return structure test, allowlist tests, non-.md domain tests

**Estimated impact:** 33 tests (~260 lines) → ~16 tests (~100 lines), covering MORE scenarios

### Not changing

- **Known filenames list** — the Set itself is well-curated
- **Subdomain escape hatch** — pragmatically correct even if theoretically debatable
- **Link tracking duplication** — 5-line duplication isn't worth restructuring
- **Function name** — `isLikelyMdFilename` is clear enough in context
- **`isFilename` field name** — renaming is bike-shedding

---

## Summary

| Metric | Before | After |
|--------|--------|-------|
| `processor.js` lines added | ~120 | ~45 |
| `test/processor.test.js` lines added | ~260 | ~100 |
| Total test count | 33 | ~16 |
| Edge cases covered | 0 (query params, fragments, empty) | 4+ |
| Unused features | 3 (confidence, context patterns, debug metadata) | 0 |
| Maintenance surface (regex patterns) | 13 | 0 |
