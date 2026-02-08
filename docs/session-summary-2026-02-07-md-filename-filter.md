# Session Summary: .md Filename False Positive Filter

**Date:** 2026-02-07
**Focus:** Detecting and filtering auto-linked .md filenames in tweet URL processing

---

## Key Decisions Made

1. **Layered detection approach** — Three layers (URL pattern → tweet context → bare domain default) rather than a single heuristic, giving confidence levels (high/medium/low) for downstream flexibility
2. **`type: 'filename-reference'` instead of dropping links** — Filtered links are preserved in the data with metadata, just skipping the expensive content fetch. This maintains an audit trail and lets the AI categorizer see the filename reference
3. **Integration point: after `expandTcoLink()`, before type classification** — The cheapest possible location. Zero additional HTTP requests — saves 1 GET + up to 50KB per false positive
4. **Allowlist for legitimate .md domains** — `obsidian.md` and subdomains are the only known `.md` TLD domains tech tweeters would intentionally link to
5. **ALL_CAPS detection deferred** — URL hostnames are normalized to lowercase by the WHATWG URL spec, so ALL_CAPS pattern detection requires bird CLI's `entities.urls[].display_url` (future enhancement)

## Files Modified

| File | Changes |
|------|---------|
| `src/processor.js` | Added `isLikelyMdFilename()` function (+~120 lines), integrated at link processing loop (~line 988) |
| `test/processor.test.js` | Added 33 new tests across 9 test groups for `isLikelyMdFilename()`, updated import |
| `docs/research-md-filename-detection.md` | Combined research document summarizing all three agent findings |
| `docs/research-synthesis-md-filename-detection.md` | Detailed synthesis with full function code and test cases |

## Research Files Created (in `.state/`)

| File | Agent | Content |
|------|-------|---------|
| `.state/research-pattern-heuristic.md` | pattern-researcher | 40+ known filenames catalog, .md TLD landscape, function design |
| `.state/research-context-analysis.md` | context-researcher | Tweet text analysis, bird `entities.urls` discovery, API data flow |
| `.state/research-http-validation.md` | http-researcher | HTTP signal analysis, parking page patterns, performance modeling |
| `.state/research-synthesis.md` | team-lead | Combined design with layered architecture |

## Key Discoveries

1. **Bird CLI discards `entities.urls[].display_url`** — Twitter's raw API response contains the original text before auto-linking (e.g., `"display_url": "memory.md"`). Bird's type system only extracts `entities.media`, dropping URL entities. This is the most reliable signal for a future enhancement.
2. **`plan.md` is a real Romanian WordPress site** — HTTP validation alone is insufficient because some `.md` domains serve legitimate content. The problem is intent ("did the user mean to link here?"), not existence.
3. **URL hostname normalization** — `new URL('https://CLAUDE.md/').hostname` returns `"claude.md"` (lowercased). ALL_CAPS detection from URLs is impossible without the original text.

## Testing Performed

- **Unit tests:** 122 total (33 new), all passing
- **Real data verification:** Tested `isLikelyMdFilename()` against all pending bookmark files:
  - `.state/pending-bookmarks.json` (35 bookmarks, 1 .md false positive: `plan.md`)
  - `.state/pending-bookmarks-no-threads.json` (20 bookmarks, 1 .md false positive: `memory.md`)
  - `.state/temp-copies/pending-bookmarks feb-copy.json` (3 bookmarks, 1 .md false positive: `memory.md`)
  - **Result:** 62 total links scanned, 3 .md domain links found, all 3 correctly FILTERED, 0 false flags on legitimate links

## PR Status

- Commit `a24d741` on `dev` branch contains all changes
- Branch `feat/md-filename-filter` created off `main` with only `src/processor.js` and `test/processor.test.js` (excluding docs)
- PR ready to create from `feat/md-filename-filter` → `main`

## Git Workflow Notes

- Used `git checkout <commit> -- <files>` approach to create a clean PR branch with only the 2 JS files (not the research docs)
- Cherry-pick (`git cherry-pick <hash>`) is the go-to for clean commits; file-checkout is for surgical PRs with subset of files

## Unfinished Work / Future Enhancements

1. **Bird CLI `entities.urls` enhancement** — Update bird to include URL entities in `TweetData`, particularly `display_url`. This would make .md filename detection trivially accurate. Changes needed in:
   - `~/projects/bird/src/lib/twitter-client-types.ts` — Add `urls` to `GraphqlTweetResult.entities` and `urlEntities` to `TweetData`
   - `~/projects/bird/src/lib/twitter-client-utils.ts` — Extract URL entities in `mapTweetResult()`
2. **Post-fetch parking page detection** — Layer 3 content validation (checking for "domain is for sale", `<!--CSTDT_3-->`, etc.) was designed but not implemented since Layer 1 catches all current cases
3. **Allowlist maintenance** — If new legitimate `.md` domains emerge in tech Twitter, add them to `LEGITIMATE_MD_DOMAINS` in `processor.js`

---

## Statistics

- ~120 lines added to `src/processor.js`
- ~260 lines added to `test/processor.test.js`
- 3 agent research reports generated
- 1 synthesis document
- 1 combined research doc for review
- 33 new unit tests
- 0 regressions
