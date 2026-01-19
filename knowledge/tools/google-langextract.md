---
title: langextract
repo: google/langextract
url: https://github.com/google/langextract
stars: 22280
language: Python
topics: [gemini, gemini-ai, gemini-api, gemini-flash, gemini-pro, information-extration, large-language-models, llm, nlp, python, structured-data]
bookmarked: 2026-01-15
category: tool
tags: [[llm]] [[information-extraction]] [[structured-data]] [[nlp]] [[gemini]] [[python]] [[source-grounding]]
---

# LangExtract

**A Python library for extracting structured information from unstructured text using LLMs with precise source grounding and interactive visualization**

Google's open-source library that uses LLMs to extract structured information from unstructured text documents based on user-defined instructions.

## Why LangExtract?

1. **Precise Source Grounding**: Maps every extraction to its exact location in the source text, enabling visual highlighting for easy traceability and verification
2. **Reliable Structured Outputs**: Enforces a consistent output schema based on your few-shot examples, leveraging controlled generation in supported models
3. **Optimized for Long Documents**: Overcomes the "needle-in-a-haystack" challenge using text chunking, parallel processing, and multiple passes for higher recall
4. **Interactive Visualization**: Generates self-contained, interactive HTML files to visualize and review thousands of extracted entities in context
5. **Flexible LLM Support**: Works with cloud-based LLMs (Google Gemini family) and local open-source models via Ollama
6. **Adaptable to Any Domain**: Define extraction tasks for any domain using just a few examples, no fine-tuning required
7. **Leverages LLM World Knowledge**: Use precise prompts and examples to influence how the extraction task utilizes LLM knowledge

## Installation

```bash
pip install langextract
```

## Quick Start

### 1. Define Your Extraction Task

```python
import langextract as lx
import textwrap

# Define the prompt and extraction rules
prompt = textwrap.dedent("""
    Extract characters, emotions, and relationships in order of appearance.
    Use exact text for extractions. Do not paraphrase or overlap entities.
    Provide meaningful attributes for each entity to add context.""")

# Provide a high-quality example to guide the model
examples = [
    lx.data.ExampleData(
        text="ROMEO. But soft! What light through yonder window breaks? It is the east, and Juliet is the sun.",
        extractions=[
            lx.data.Extraction(
                extraction_class="character",
                extraction_text="ROMEO",
                attributes={"emotional_state": "wonder"}
            ),
            lx.data.Extraction(
                extraction_class="emotion",
                extraction_text="But soft!",
                attributes={"feeling": "gentle awe"}
            ),
            lx.data.Extraction(
                extraction_class="relationship",
                extraction_text="Juliet is the sun",
                attributes={"type": "metaphor"}
            ),
        ]
    )
]
```

### 2. Extract from Text

```python
# Initialize extractor
extractor = lx.Extractor(
    prompt=prompt,
    examples=examples,
    model="gemini-2.0-flash"  # or use Ollama for local models
)

# Extract from document
result = extractor.extract(document_text)

# Access extractions
for extraction in result.extractions:
    print(f"{extraction.extraction_class}: {extraction.extraction_text}")
    print(f"Attributes: {extraction.attributes}")
```

### 3. Visualize Results

```python
# Generate interactive HTML visualization
lx.visualization.create_html(
    result,
    output_path="extractions.html"
)
```

## Use Cases

- **Clinical Notes**: Extract medications, diagnoses, symptoms from medical records
- **Radiology Reports**: Structure findings from unstructured reports (RadExtract)
- **Literature Analysis**: Extract characters, emotions, themes from texts
- **Legal Documents**: Extract entities, dates, obligations from contracts
- **Research Papers**: Extract methodologies, findings, citations

## Key Features

- **Few-shot learning**: Define tasks with just a few examples
- **Source traceability**: Every extraction maps to exact text location
- **Attribute extraction**: Add context and metadata to each entity
- **Long document support**: Efficient processing of large texts
- **Interactive visualization**: Review and verify extractions visually
- **Multi-model support**: Gemini, OpenAI, or local models via Ollama

## Model Support

- **Cloud Models**: Google Gemini family (Flash, Pro)
- **OpenAI**: GPT-4, GPT-3.5
- **Local Models**: Any model via Ollama integration
- **Custom Providers**: Extensible to add your own model providers

## Example Domains

### Medical (RadExtract)
Extract structured findings from radiology reports with precise source grounding

### Literary Analysis
Extract characters, emotions, relationships from classic literature (Romeo and Juliet examples)

### Medication Extraction
Parse clinical notes to extract medication names, dosages, frequencies with attributes

## Community

The library supports community providers for additional model integrations and custom extraction workflows.
