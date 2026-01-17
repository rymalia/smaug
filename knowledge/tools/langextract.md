# LangExtract

**Repository:** [google/langextract](https://github.com/google/langextract)
**Stars:** 21,380
**Language:** Python
**Source:** GitHub API

## Overview

LangExtract is a Python library that uses LLMs to extract structured information from unstructured text documents based on user-defined instructions. It processes materials such as clinical notes or reports, identifying and organizing key details while ensuring the extracted data corresponds to the source text.

## Key Features

1. **Precise Source Grounding:** Maps every extraction to its exact location in the source text, enabling visual highlighting for easy traceability and verification.

2. **Reliable Structured Outputs:** Enforces a consistent output schema based on your few-shot examples, leveraging controlled generation in supported models like Gemini to guarantee robust, structured results.

3. **Optimized for Long Documents:** Overcomes the "needle-in-a-haystack" challenge of large document extraction by using an optimized strategy of text chunking, parallel processing, and multiple passes for higher recall.

4. **Interactive Visualization:** Instantly generates a self-contained, interactive HTML file to visualize and review thousands of extracted entities in their original context.

5. **Flexible LLM Support:** Supports your preferred models, from cloud-based LLMs like the Google Gemini family to local open-source models via the built-in Ollama interface.

6. **Adaptable to Any Domain:** Define extraction tasks for any domain using just a few examples. LangExtract adapts to your needs without requiring any model fine-tuning.

## Topics

gemini, gemini-ai, gemini-api, gemini-flash, gemini-pro, information-extration, large-language-models, llm, nlp, python, structured-data

## Use Cases

- Clinical note structuring and medical data extraction
- Legal document processing
- Research literature analysis
- Financial report parsing
- Any domain requiring extraction of structured data from unstructured text

## Bookmarked By

@simplifyinAI on January 15, 2026 - Noted as Google's open-sourced solution with precise source grounding and interactive visualization.
