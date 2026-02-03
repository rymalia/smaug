---
title: "Dash: Open-Source SQL Data Agent"
type: article
date_added: 2026-02-01
source: "https://x.com/i/article/2018059495335764273"
author: "Ashpreet Bedi"
tags: [data-agents, sql, llm, openai, self-learning]
via: "Twitter bookmark from @ashpreetbedi"
---

OpenAI's in-house data agent represents a breakthrough in enterprise AI. Ashpreet Bedi has open-sourced a similar implementation called Dash, which uses 6 layers of context to ground its answers and improve with every run.

## The 6 Layers of Context

1. **Table Usage** - schema, columns, relationships
2. **Human Annotations** - metrics, definitions, gotchas
3. **Query Patterns** - SQL that's known to work
4. **Institutional Knowledge** - external docs, research
5. **Memory** - error patterns, discovered fixes
6. **Runtime Context** - live schema when things change

## Key Features

- Self-learning loop: static knowledge + continuous learning
- GPU-poor learning approach (5 lines of code to enable)
- Hybrid search for context retrieval at runtime
- Extensive evaluation suite (string matching, LLM grading, golden SQL)
- Ships with UI via Agno platform
- F1 dataset example included

## Quick Start

```bash
git clone https://github.com/agno-agi/dash && cd dash
cp example.env .env  # Add OPENAI_API_KEY
docker compose up -d --build
docker exec -it dash-api python -m dash.scripts.load_data
docker exec -it dash-api python -m dash.scripts.load_knowledge
```

## Key Takeaways

- Data agents are one of the best enterprise use cases for AI right now
- Every company over a certain size should have one
- Context is everything - stateless agents repeat mistakes forever
- Continuous learning without fine-tuning or retraining is possible

## Links

- [GitHub Repository](https://github.com/agno-agi/dash)
- [OpenAI's Data Agent Post](https://openai.com/index/inside-our-in-house-data-agent/)
- [Ashpreet's Previous Work](https://www.ashpreetbedi.com/articles/sql-agent)
- [Original Tweet/Article](https://x.com/i/article/2018059495335764273)
