---
title: "Dash"
type: tool
date_added: 2026-02-02
source: "https://github.com/agno-agi/dash"
tags: ["agents", "data-agent", "sql", "self-learning", "ai"]
via: "Twitter bookmark from @ashpreetbedi"
---

Dash is a self-learning data agent inspired by OpenAI's in-house data agent implementation. It improves with every query through a novel architecture that combines grounded context with continuous learning—without requiring fine-tuning.

## The Problem

Traditional Text-to-SQL agents fail in production because:
- Schemas lack business meaning and context
- Types are misleading or ambiguous
- Tribal knowledge and gotchas are missing
- No memory: agents repeat the same mistakes across sessions
- Results lack interpretation and actionable insights

Dash solves this with **6 layers of context** and a **self-learning loop**.

## Key Features

### 6 Layers of Context
1. **Table Usage**: Schema, columns, relationships
2. **Human Annotations**: Metrics, definitions, business rules
3. **Query Patterns**: SQL that is known to work
4. **Institutional Knowledge**: External docs, research, wikis
5. **Memory**: Error patterns and discovered fixes
6. **Runtime Context**: Live schema changes

### Self-Learning Loop
- **Static Knowledge**: Curated queries and business context maintained by your team
- **Continuous Learning**: Patterns discovered through trial and error ("gpu-poor continuous learning")
- Never repeats mistakes: when a query fails, it saves the fix as a reusable learning

### Results as Insights, Not Just Rows
Dash reasons about what makes an answer useful. Instead of just `Hamilton: 11`, it returns: "Lewis Hamilton dominated 2019 with 11 wins out of 21 races, more than double Bottas's 4 wins. This secured his sixth world championship."

## Quick Start

```bash
git clone https://github.com/agno-agi/dash && cd dash
cp example.env .env  # Add OPENAI_API_KEY

docker compose up -d --build

# Load sample data and knowledge
docker exec -it dash-api python -m dash.scripts.load_data
docker exec -it dash-api python -m dash.scripts.load_knowledge

# Navigate to http://localhost:8000/docs
```

Then connect to the UI:
1. Open [os.agno.com](https://os.agno.com)
2. Add OS → Local → `http://localhost:8000`
3. Click Connect

Try on sample F1 dataset:
- Who won the most F1 World Championships?
- How many races has Lewis Hamilton won?
- Compare Ferrari vs Mercedes points 2015-2020

## Architecture Philosophy

The key insight: context is everything. Raw LLMs hallucinate column names, miss type quirks, and ignore domain knowledge. Dash retrieves relevant context at query time via hybrid search, uses this grounded foundation to generate SQL, then interprets results through the lens of business context.

## Links

- [GitHub](https://github.com/agno-agi/dash)
- [OpenAI's In-House Data Agent Post](https://openai.com/index/inside-our-in-house-data-agent/)
- [Ashpreet's Previous Work](https://www.ashpreetbedi.com/articles/sql-agent)
- [Original Tweet](https://x.com/ashpreetbedi/status/2018479845886320728)
