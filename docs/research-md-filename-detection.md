# Research: .md Filename vs Domain Detection

**Date:** 2026-02-07
**Methodology:** Agent team with three parallel research tracks
**Status:** Research complete, implementation in progress

---

## Problem Statement

Twitter auto-links text like `CLAUDE.md`, `Plan.md`, `memory.md` in tweets, wrapping them in `t.co` shortened URLs that resolve to Moldova's `.md` ccTLD domains. Smaug then expands these, classifies them as `type: 'article'`, and fetches up to 50KB of irrelevant HTML from unrelated websites.

### Real Cases Found in Data

| Tweet Author | Filename Referenced | Expanded To | What Was Fetched |
|---|---|---|---|
| @threepointone | `plan.md` (coding scratch file) | `https://plan.md/` | 50KB Romanian WordPress site (lang="ro-RO") |
| @alex_prompter | `memory.md` (AI scratchpad file) | `https://memory.md/` | Parked domain: "The content of the page cannot be displayed" |

---

## Research Track 1: Pattern-Based Heuristic

**Agent:** pattern-researcher
**Full report:** `.state/research-pattern-heuristic.md`

### Key Findings

1. **`.md` is Moldova's ccTLD** — ~12,000+ registered domains, open registration, ~$39/year
2. **Only one legitimate `.md` domain matters for tech Twitter:** `obsidian.md` (the note-taking app) and its subdomains
3. **40+ known dev filenames cataloged** across three categories:
   - Standard project files: README, CHANGELOG, CONTRIBUTING, LICENSE, SECURITY, etc.
   - AI/coding tool files (2025-2026): CLAUDE, MEMORY, PLAN, AGENTS, RULES, INSTRUCTIONS, etc.
   - Documentation: setup, install, guide, docs, api, config, etc.
4. **The `.md` TLD is also marketed for medical professionals** ("MD" = Medical Doctor), but these domains are unlikely in tech tweets

### Proposed Approach
- Maintain a `Set` of known dev filename stems (case-insensitive)
- Check for ALL_CAPS naming pattern (`/^[A-Z][A-Z_0-9]+$/`) as a secondary signal
- Allowlist legitimate `.md` domains (obsidian.md and subdomains)
- Only filter bare domains with no path — `plan.md/some/page` passes through

---

## Research Track 2: Tweet Context Analysis

**Agent:** context-researcher
**Full report:** `.state/research-context-analysis.md`

### Key Findings

1. **Twitter replaces original text:** The `text` field from bird CLI contains `t.co` links, NOT the original filenames. We can't directly see "memory.md" — it's been replaced with `https://t.co/YSz85YYwut`.

2. **Bird discards the best signal:** Twitter's raw GraphQL API response includes `entities.urls[].display_url` which contains the *original text the user typed*:
   ```json
   {
     "url": "https://t.co/YSz85YYwut",
     "expanded_url": "https://memory.md/",
     "display_url": "memory.md"        // ← What the user ACTUALLY TYPED
   }
   ```
   Bird's type system (`GraphqlTweetResult`) only extracts `entities.media`, dropping URL entity data. This is the single most reliable signal available but requires a bird CLI change.

3. **Surrounding text still provides strong signals:** Words like "files", "save as", "cache in", possessive pronouns before links, multiple `.md` links in one tweet.

4. **Multiple `.md` links in one tweet = very strong signal:** @threepointone's tweet has three t.co links that all resolve to `.md` domains — clearly a list of filenames.

### Proposed Approach (Two Strategies)
- **Short-term (Smaug-only):** Regex-based tweet text analysis for file-related context words
- **Long-term (bird change):** Add `entities.urls` to bird's `TweetData` type, making detection trivial via `display_url`

---

## Research Track 3: HTTP Response Validation

**Agent:** http-researcher
**Full report:** `.state/research-http-validation.md`

### Key Findings

1. **HTTP validation alone is insufficient:** `plan.md` IS a legitimate WordPress site with 50KB of real content. No HTTP signal can distinguish it from any other real website. The problem isn't "is this site real?" — it's "did the tweet author intend to link here?"

2. **Parking page detection works for dead domains:** `memory.md` returns "The content of the page cannot be displayed" with a hosting provider marker `<!--CSTDT_3-->`. Recognizable patterns include: "domain is for sale", "buy this domain", "parked by", "sedoparking", etc.

3. **`expandTcoLink()` wastes useful data:** The HEAD request returns a full response object with headers and status code, but only `response.url` is kept. The rest is discarded.

4. **URL-only pattern is the strongest HTTP-adjacent signal:** A domain with no path (`https://plan.md/` vs `https://plan.md/some-article`) strongly suggests auto-linking.

5. **Performance:** Pre-filtering before `fetchContent()` saves 1 GET request + up to 50KB download per false positive. All proposed checks cost zero additional HTTP requests.

### Proposed Approach
- Layer 1: URL pattern (bare domain, no path) — zero cost
- Layer 2: Post-fetch parking page content detection — zero extra cost (content already fetched)
- Layer 3: Enhanced `expandTcoLink()` returning metadata — zero extra cost (reuses existing HEAD)

---

## Unified Solution Design

### Architecture: Three Layers (Fast → Medium → Slow)

```
Expanded URL
    │
    ▼
┌─────────────────────────────────────────┐
│ LAYER 1: URL Pattern Check (0ms)        │
│ - Is it a bare .md TLD domain?          │
│ - Is the domain stem a known filename?  │
│ - Is it ALL_CAPS pattern?               │
│ - Allowlist: obsidian.md passes through │
│ → Catches ~95% of cases                 │
└──────────────┬──────────────────────────┘
               │ uncertain?
               ▼
┌─────────────────────────────────────────┐
│ LAYER 2: Tweet Context Check (0ms)      │
│ - "files", "save as", "cache in"        │
│ - "markdown", "repo", "project"         │
│ - AI tool names: cursor, claude code    │
│ → Medium confidence for unknown names   │
└──────────────┬──────────────────────────┘
               │ still uncertain?
               ▼
┌─────────────────────────────────────────┐
│ LAYER 3: Default for bare .md domains   │
│ - Low confidence filter for unknown     │
│   bare .md domains (the overwhelmingly  │
│   common case in tech Twitter)          │
└─────────────────────────────────────────┘
```

### Integration Point

`processor.js` line ~866, between `expandTcoLink()` resolution and type classification. Detected filenames get `type: 'filename-reference'` and skip content fetching.

### Performance Impact

| Metric | Before | After |
|--------|--------|-------|
| HTTP requests per .md false positive | 2 (HEAD + GET) | 1 (HEAD only) |
| Content downloaded per false positive | Up to 50KB | 0 bytes |
| Processing time per false positive | ~2-30 seconds | ~0ms |

### Future Enhancement

Update bird CLI to include `entities.urls[].display_url` from Twitter's API. This gives the exact text the user typed before Twitter auto-linked it, making detection trivially accurate without heuristics.

---

## Detailed Research Files

- `.state/research-pattern-heuristic.md` — Full pattern analysis, filename catalog, TLD landscape
- `.state/research-context-analysis.md` — Tweet text analysis, bird entities discovery, API data flow
- `.state/research-http-validation.md` — HTTP signal analysis, parking page patterns, performance modeling
- `.state/research-synthesis.md` — Combined design with full function code and test cases
