---
title: json-render
repo: vercel-labs/json-render
url: https://github.com/vercel-labs/json-render
stars: 6825
language: TypeScript
topics: []
bookmarked: 2026-01-14
category: tool
tags: [[ai]] [[ui-generation]] [[json]] [[react]] [[guardrails]] [[streaming]]
---

# json-render

**AI → JSON → UI**

Let end users generate dashboards, widgets, apps, and data visualizations from prompts — safely constrained to components you define.

## Why json-render?

When users prompt for UI, you need guarantees. json-render gives AI a **constrained vocabulary** so output is always predictable:

- **Guardrailed** — AI can only use components in your catalog
- **Predictable** — JSON output matches your schema, every time
- **Fast** — Stream and render progressively as the model responds

## Installation

```bash
npm install @json-render/core @json-render/react
```

## Quick Start

### 1. Define Your Catalog

```typescript
import { createCatalog } from '@json-render/core';
import { z } from 'zod';

const catalog = createCatalog({
  components: {
    Card: {
      props: z.object({ title: z.string() }),
      hasChildren: true,
    },
    Metric: {
      props: z.object({
        label: z.string(),
        valuePath: z.string(),      // Binds to your data
        format: z.enum(['currency', 'percent', 'number']),
      }),
    },
    Button: {
      props: z.object({
        label: z.string(),
        action: ActionSchema,        // AI declares intent, you handle it
      }),
    },
  },
  actions: {
    export_report: { description: 'Export dashboard to PDF' },
    refresh_data: { description: 'Refresh all metrics' },
  },
});
```

### 2. Register Your Components

```tsx
const registry = {
  Card: ({ element, children }) => (
    <div className="card">
      <h3>{element.props.title}</h3>
      {children}
    </div>
  ),
  Metric: ({ element }) => {
    const value = useDataValue(element.props.valuePath);
    return <div className="metric">{format(value)}</div>;
  },
  Button: ({ element, onAction }) => (
    <button onClick={() => onAction(element.props.action)}>
      {element.props.label}
    </button>
  ),
};
```

### 3. Let AI Generate

```tsx
import { DataProvider, ActionProvider, Renderer, useUIStream } from '@json-render/react';

function Dashboard() {
  const { tree, send } = useUIStream({ api: '/api/generate' });

  return (
    <DataProvider initialData={{ revenue: 125000, growth: 0.15 }}>
      <ActionProvider actions={{
        export_report: () => downloadPDF(),
        refresh_data: () => refetch(),
      }}>
        <input
          placeholder="Create a revenue dashboard..."
          onKeyDown={(e) => e.key === 'Enter' && send(e.target.value)}
        />
        <Renderer tree={tree} components={registry} />
      </ActionProvider>
    </DataProvider>
  );
}
```

## Key Features

### Conditional Visibility

Show/hide components based on data, auth, or complex logic:

```json
{
  "type": "Alert",
  "props": { "message": "Error occurred" },
  "visible": {
    "and": [
      { "path": "/form/hasError" },
      { "not": { "path": "/form/errorDismissed" } }
    ]
  }
}
```

### Rich Actions

Actions with confirmation dialogs and callbacks:

```json
{
  "type": "Button",
  "props": {
    "label": "Refund Payment",
    "action": {
      "name": "refund",
      "params": {
        "paymentId": { "path": "/selected/id" }
      },
      "confirm": {
        "title": "Confirm Refund",
        "message": "Refund ${/refund/amount} to customer?",
        "variant": "danger"
      },
      "onSuccess": { "set": { "/ui/success": true } }
    }
  }
}
```

### Built-in Validation

```json
{
  "type": "TextField",
  "props": {
    "label": "Email",
    "valuePath": "/form/email",
    "checks": [
      { "fn": "required", "message": "Email is required" },
      { "fn": "email", "message": "Invalid email" }
    ]
  }
}
```

## Packages

- `@json-render/core` - Types, schemas, visibility, actions, validation
- `@json-render/react` - React renderer, providers, hooks

## Use Cases

- User-generated dashboards
- Custom report builders
- Dynamic admin panels
- Prompt-driven widgets
- Interactive data visualizations

## How It Works

```
User Prompt → AI + Catalog → JSON Tree → React Components → Interactive UI
             (guardrailed)  (predictable)
```

The AI can only reference components in your catalog, ensuring safe, deterministic output that renders exactly as expected.
