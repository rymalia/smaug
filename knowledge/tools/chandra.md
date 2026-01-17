# Chandra - OCR Model for Complex Documents

**Repository:** [datalab-to/chandra](https://github.com/datalab-to/chandra)
**Language:** Python
**Stars:** 4,469
**License:** Apache 2.0 & OpenRAIL-M

## Overview

Chandra is a state-of-the-art OCR (Optical Character Recognition) model developed by Datalab that specializes in handling complex documents with multiple content types and challenging layouts.

## Key Capabilities

- **Handwriting Recognition** - Reads cursive and messy print that challenges traditional OCR systems (doctor notes, filled forms, homework)
- **Table Preservation** - Maintains table structure including merged cells (colspan/rowspan) in financial filings, invoices, and data tables
- **Math Equations** - Renders inline and block equations as LaTeX for textbooks, worksheets, research papers
- **Form Processing** - Reconstructs checkboxes, radio buttons, and form field values
- **Complex Layouts** - Handles multi-column documents, newspapers, textbooks with figures and captions
- **40+ Languages** - Supports multiple language inputs

## Installation

```bash
pip install chandra-ocr
```

For better performance with HuggingFace inference, install flash attention:
```bash
# Recommended for HuggingFace mode
```

From source:
```bash
git clone https://github.com/datalab-to/chandra.git
cd chandra
uv sync
source .venv/bin/activate
```

## Usage

### Command Line Interface

```bash
# Single file with vLLM server
chandra input.pdf ./output --method vllm

# Directory with local model
chandra ./documents ./output --method hf

# Interactive web app
chandra_app

# Start vLLM server first
chandra_vllm
```

### Python API

```python
from chandra.model import InferenceManager
from chandra.input import load_pdf_images

manager = InferenceManager(method="hf")
images = load_pdf_images("document.pdf")
results = manager.generate(images)
print(results[0].markdown)
```

## Inference Methods

- **HuggingFace Transformers** - Run locally for development and testing
- **vLLM Server** - Deploy for production throughput and API access

## Output Formats

- **Markdown** - Clean, readable text with layout structure
- **HTML** - Web-ready formatted content
- **JSON** - Full layout metadata with bounding box coordinates

## Benchmarks

Performs competitively on the olmocr bench for OCR accuracy across document types.

## Community

Join the [Discord](https://discord.gg/KuZwXNGnfH) server for development discussion and support.

## Hosted API

A hosted API with additional accuracy improvements is available at [datalab.to](https://www.datalab.to/) with a free playground for testing.

## Relevant Bookmarks

- Compared against DeepSeek OCR, Qwen3-VL, Dots OCR, and Granite Docling in open-source OCR evaluation
- Featured as the best open-source OCR solution in comparative analysis (Jan 13, 2026)
