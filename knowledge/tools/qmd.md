# QMD - Quick Markdown Search

A hybrid search engine for local knowledge bases that combines BM25 full-text search, vector semantic search, and LLM re-ranking—all running locally via node-llama-cpp with GGUF models.

## Overview

QMD is designed as a foundational tool for personal knowledge management and AI agent integration. It indexes markdown notes, documentation, meeting transcripts, and knowledge bases for efficient retrieval using multiple search strategies.

## Key Features

- **Hybrid Search Pipeline**: Combines BM25 keyword search, vector embeddings, and query expansion
- **Local Inference**: All processing happens on-device with no external API calls
- **Query Expansion**: Uses Qwen 3 model to generate alternative query formulations for better coverage
- **Ranked Reciprocal Fusion (RRF)**: Intelligently combines multiple search results with configurable weighting
- **MCP Server Integration**: Exposes tools for Claude Desktop and Claude Code integration
- **Agent-Optimized Output**: JSON and file-list formats designed for agentic workflows

## Search Commands

- `qmd search "query"` - Fast BM25 keyword search (with collection filter support)
- `qmd vsearch "query"` - Semantic vector search (with collection filter support)
- `qmd query "query"` - Hybrid search with reranking (best quality)
- `qmd get "path or docid"` - Retrieve specific document
- `qmd multi-get "pattern"` - Retrieve multiple documents by glob pattern

## MCP Tools Exposed

- `qmd_search` - BM25 keyword search
- `qmd_vsearch` - Semantic vector search
- `qmd_query` - Hybrid search with reranking
- `qmd_get` - Document retrieval by path or docid
- `qmd_multi_get` - Batch document retrieval
- `qmd_status` - Index health and collection info

## Architecture Highlights

1. **Query Expansion**: Original query gets 2x weight; expanded queries provide alternative coverage
2. **Reciprocal Rank Fusion**: Combines multiple ranking signals without explicitly choosing winners
3. **Recency Bonus**: Can be configured to weight recent documents higher
4. **Reranking**: Uses Qwen 3 LLM for final relevance scoring

## Use Cases

- Personal knowledge base for quick retrieval
- Meeting transcript search and summarization
- Documentation query interface
- AI agent context retrieval for agentic flows
- Time-aware retrieval systems (via RRF customization)

## Links

- **Repository**: https://github.com/tobi/qmd
- **Creator**: Tobi Lütke (@tobi)

## Notes

While QMD is excellent for static documentation, it's optimized for on-device performance and can be customized via RRF formula adjustments for domain-specific use cases (like adding recency bonuses for time-sensitive retrieval).
