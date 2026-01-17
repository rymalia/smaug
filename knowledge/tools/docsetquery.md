---
title: DocSetQuery
author: PaulSolt
url: https://github.com/PaulSolt/DocSetQuery
stars: 63
language: Python
bookmarked: Thursday, January 15, 2026
---

# DocSetQuery

Tooling for Agents to Create Markdown Documentation from DocSet Bundles.

## Overview

Local-first Apple documentation extraction, cleanup, and search. Built for fast developer lookup and agent workflows that need deterministic, citeable Markdown instead of scraping the web.

## Why This Exists

- Apple docs are large and dynamic; agents need stable, local references
- DocC exports are noisy; need predictable front matter and trimmed tables of contents
- Local search should be instant, without re-reading docsets for every query

## What You Get

| Tool | Purpose |
|------|---------|
| `tools/docset_query.py` | Exports DocC content from Apple docset to Markdown |
| `tools/docset_sanitize.py` | Rebuilds front matter + trims TOC for cleaner context |
| `tools/docindex.py` | Builds local JSON index for fast search by heading/key sections |
| `tools/docmeta.py` | Peeks front matter/TOC quickly for debugging |
| `scripts/sync_docs.sh` | Syncs canonical docs cache into `docs/apple` |

## Quickstart

```bash
# Export a framework/topic tree to Markdown
python tools/docset_query.py export \
  --root /documentation/vision \
  --output docs/apple/vision.md

# Sanitize the export (trim TOC, rebuild front matter)
python tools/docset_sanitize.py --input docs/apple/vision.md --in-place --toc-depth 2

# Build or refresh the search index
python tools/docindex.py rebuild

# Search headings/key sections
python tools/docindex.py search "CVPixelBuffer"
```

## Agent Workflow

This is the recommended flow for agent-driven Apple development:

1. **Search locally first** — Agents call `docindex.py search` against `docs/apple`
2. **Fetch only what's missing** — If topic not found, use `docset_query.py fetch` or `export`
3. **Sanitize for stable context** — Run `docset_sanitize.py` to keep front matter and TOC consistent
4. **Rebuild the index** — `docindex.py rebuild` keeps agent search fast and deterministic
5. **Keep a canonical cache** — Sync with `scripts/sync_docs.sh` so `docs/apple` stays lightweight without committing full docset

This approach lets agents answer questions with local, vetted Markdown and avoids hitting remote docs during runs.

## How It Works

### Docset Export (`tools/docset_query.py`)

- Reads the Dash Apple API Reference docset directly (SQLite + brotli chunks)
- Commands:
  - `export` — walk a documentation tree and emit a single Markdown file
  - `fetch` — render a single symbol/topic (optionally to stdout)
  - `init` — prebuild manifests for faster traversal

**Defaults:**
- Docset path: `~/Library/Application Support/Dash/DocSets/Apple_API_Reference/Apple_API_Reference.docset`
- Language: `swift`
- Cache: `~/.cache/apple-docs`
- `export` depth: 7, `fetch` depth: 1

**Overrides:**
- `--docset` or `DOCSET_ROOT` for alternate docsets
- `--language` for alternate language variants
- `DOCSET_CACHE_DIR` for cache location

### Sanitize Exports (`tools/docset_sanitize.py`)

- Rebuilds front matter with stable summary and key sections
- Trims TOC depth and drops noisy stopwords (e.g., "discussion", "parameters")
- Keeps output deterministic so agent prompts stay consistent

### Index and Search (`tools/docindex.py`)

- Builds `Build/DocIndex/index.json` from Markdown in `docs/apple`
- Indexes front matter, headings, and key sections
- Search matches headings/key sections and returns anchored paths

### Sync Docs Cache (`scripts/sync_docs.sh`)

- `docs/apple` is cache-only and `.gitignore`'d
- Pull: `DOCS_SOURCE=~/docs/apple scripts/sync_docs.sh pull --allow-delete`
- Push: `DOCS_SOURCE=~/docs/apple scripts/sync_docs.sh push`

## Notes

- Assumes a local Apple docset; does not download docsets
- Docsets come from Dash: https://kapeli.com/dash
- Alternative docset feeds: https://github.com/Kapeli/feeds
- Scripts are CLI-first so agents can script them
- All dependencies are minimal and local-first

## Status

**Implemented:**
- Docset export (`tools/docset_query.py`): `export`, `fetch`, `init`
- Sanitizer (`tools/docset_sanitize.py`): front matter rebuild + TOC trimming
- Index + search (`tools/docindex.py`): JSON index + heading/key-section search
- Metadata peek (`tools/docmeta.py`): front matter/TOC inspection
- Cache sync (`scripts/sync_docs.sh`): pull/push to canonical docs folder

**Planned (not implemented):**
- Automated docset download/updates from Kapeli feeds or other vendor sources

## Use Cases

- **Agent documentation for Apple frameworks** — Keep agent-friendly docs local and versioned
- **Fast development loops** — Instant doc lookups during agent runs, no network calls
- **Deterministic outputs** — Agents reference same doc versions, reproducible results
- **Markdown-first workflow** — Convert Apple docs to searchable, filterable Markdown

## Related

- **Dash** - GUI docset reader with docset feeds
- **Peter Steinberger's Agent Scripts** - Complementary agent automation patterns
- **Apple API Reference** - Official Apple documentation

