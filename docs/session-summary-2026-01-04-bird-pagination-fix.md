# Session Summary: Bird Bookmarks Pagination Fix (January 4, 2026)

This document summarizes a debugging session for the bird CLI (used by Smaug). It's intended for:
- **The developer** as a reference
- **Future Claude Code sessions** as context for continuing work

---

## Session Goals

1. Investigate why `bird bookmarks -n 20` fails with "Query: Unspecified" error
2. Fix the issue so Smaug can fetch more than 15 bookmarks at a time

---

## What We Discovered

### The Problem

```bash
bird bookmarks -n 15 --json   # Works fine
bird bookmarks -n 16 --json   # ❌ Failed to fetch bookmarks: Query: Unspecified
bird bookmarks -n 20 --json   # ❌ Failed to fetch bookmarks: Query: Unspecified
```

The threshold is exactly 16 - any count >= 16 fails.

### Root Cause Analysis

1. **Twitter's GraphQL API has a per-request limit of ~15 items** for the Bookmarks endpoint
2. The `getBookmarks()` function in bird made a **single API request** with the full count
3. Unlike the `search()` function (which uses cursor-based pagination), bookmarks had no pagination support
4. When count > 15, Twitter returns an error: "Query: Unspecified"

### Key Code Locations

| File | Purpose |
|------|---------|
| `~/projects/bird/src/lib/twitter-client-timelines.ts` | Contains `getBookmarks()` - **this is what we fixed** |
| `~/projects/bird/src/lib/twitter-client-search.ts` | Contains `search()` with working pagination (used as reference) |
| `~/projects/bird/src/lib/twitter-client-utils.ts` | Contains `extractCursorFromInstructions()` for pagination |
| `~/projects/bird/src/commands/bookmarks.ts` | CLI command that calls the client |

### Additional Finding: Query ID Discovery

The runtime query ID discovery (`~/.config/bird/query-ids-cache.json`) doesn't find "Bookmarks" in Twitter's client bundles - only 9 of 11 operations are discovered. The code correctly falls back to hardcoded query IDs in `FALLBACK_QUERY_IDS`.

---

## The Fix

### File Modified: `~/projects/bird/src/lib/twitter-client-timelines.ts`

**Changed `getBookmarks()` to use pagination** (like the `search()` function does):

Key changes:
1. Added imports for `TweetData` and `extractCursorFromInstructions`
2. Set `pageSize = 15` (Twitter's apparent limit)
3. Added `cursor` variable to track pagination state
4. Wrapped API call in `fetchPage()` function that extracts cursor from response
5. Added `while (tweets.length < count)` loop to fetch multiple pages
6. Added deduplication with `Set<string>` to avoid duplicate tweets
7. Graceful handling: if pagination fails mid-way, return what we have

The fix mirrors the pagination pattern already used in `search()`.

### Testing Results

```bash
# Before fix:
bird bookmarks -n 20 --json
# ❌ Failed to fetch bookmarks: Query: Unspecified

# After fix:
cd ~/projects/bird && node dist/cli.js bookmarks -n 20 --json | jq 'length'
# 15  (returns all available bookmarks - user only has 15)

# All tests pass:
pnpm run test
# ✓ 200 passed | 14 skipped
```

---

## Current State

### What's Done
- [x] Fix implemented in local source: `~/projects/bird/src/lib/twitter-client-timelines.ts`
- [x] Local build created: `~/projects/bird/dist/`
- [x] All 200 tests pass

### What's NOT Done
- [ ] **Global bird install NOT updated** - The fix is only in the local build
- [ ] Changes not committed to git
- [ ] Changes not pushed to upstream repo

### File Locations

| Location | Status |
|----------|--------|
| `~/projects/bird/` | Local source with fix applied |
| `~/projects/bird/dist/cli.js` | Rebuilt with fix |
| `/opt/homebrew/bin/bird` | Symlink to OLD global install (no fix) |
| `/opt/homebrew/lib/node_modules/@steipete/bird/` | Global npm install (no fix) |

---

## Next Steps

### 1. Update Global bird Install

Choose one of these approaches:

```bash
# Option A: Link local build as global (recommended for development)
cd ~/projects/bird
npm link

# Option B: Reinstall globally from local source
cd ~/projects/bird
npm install -g .

# Option C: Just use local build directly
alias bird="node ~/projects/bird/dist/cli.js"
```

### 2. Test with Smaug

```bash
# After updating global bird:
cd ~/projects/smaug
npx smaug fetch 20
npx smaug run
```

### 3. Commit Changes (if desired)

```bash
cd ~/projects/bird
git add src/lib/twitter-client-timelines.ts
git commit -m "Add pagination to getBookmarks for counts > 15

Twitter's Bookmarks GraphQL endpoint has a per-request limit of ~15 items.
Previously, requesting more would fail with 'Query: Unspecified' error.

Now uses cursor-based pagination (like search) to fetch multiple pages
when the requested count exceeds the API limit.

Fixes: bird bookmarks -n 20 failing"
```

---

## Relationship: Smaug & Bird

**Important context for future sessions:**

Smaug is essentially a wrapper around bird's functionality:

```
Smaug (bookmark archiver)
    └── processor.js calls bird CLI
        └── bird bookmarks -n X --json
        └── bird read <tweet_id> --json
        └── bird likes -n X --json
```

Any issues with fetching/parsing tweets are likely bird bugs, not smaug bugs. When debugging smaug fetch problems:
1. First test the bird command directly
2. Fix in bird source (`~/projects/bird/src/`)
3. Rebuild bird (`pnpm run build:dist`)
4. Update global install or use local build
5. Then test smaug again

---

## Quick Reference Commands

```bash
# Test bird directly (local build)
cd ~/projects/bird && node dist/cli.js bookmarks -n 20 --json | jq 'length'

# Test bird directly (global install)
bird bookmarks -n 20 --json | jq 'length'

# Rebuild bird after changes
cd ~/projects/bird && npx pnpm run build:dist

# Run bird tests
cd ~/projects/bird && npx pnpm run test

# Check which bird is being used
which bird
bird --version

# Check global npm packages
npm list -g @steipete/bird
```

---

## Session Metadata

- **Date:** January 4, 2026
- **Duration:** ~20 minutes
- **Model:** Claude Opus 4.5
- **Working Directories:**
  - `/Users/rymalia/projects/smaug` (primary)
  - `/Users/rymalia/projects/bird` (for the fix)
