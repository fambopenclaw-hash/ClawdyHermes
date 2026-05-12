---
name: create-markdown
description: Take any file input (PDF, image, text, code, CSV, HTML, markdown, audio, etc.), extract/analyze its content, and generate a comprehensive structured Markdown report saved to the Obsidian vault with a _hermes suffix. Handles PDF extraction (text + OCR), image analysis, code parsing, tabular data, and plain text files.
tags: [obsidian, analysis, report, pdf, image, code, csv, markdown]
---

# Create Markdown Reports from Any File

## Overview

Take **any file** the user provides → extract its content using the appropriate method → generate a structured Markdown report → save to Obsidian vault with `_hermes` suffix.

**Supported file types:**
| Type | Extensions | Extraction Method |
|------|-----------|-------------------|
| PDF | `.pdf` | Text extraction via `scripts/extract_file.py` (pdfplumber → PyMuPDF+Tesseract OCR) |
| Images | `.png`, `.jpg`, `.jpeg`, `.webp`, `.gif`, `.bmp`, `.svg` | Vision analysis (`vision_analyze` tool) |
| Text/Markdown | `.txt`, `.md`, `.log`, `.rst`, `.tex` | Direct file read |
| Code | `.py`, `.js`, `.ts`, `.jsx`, `.tsx`, `.html`, `.css`, `.json`, `.yaml`, `.yml`, `.toml`, `.xml`, `.sh`, `.rb`, `.php`, `.go`, `.rs`, `.java`, `.c`, `.cpp`, `.h`, `.sql`, `.r`, `.pl`, `.lua` | Direct file read + language detection |
| Tabular | `.csv`, `.tsv`, `.xlsx`, `.xls` | Python parsing (pandas/csv) |
| HTML | `.html`, `.htm` | HTML-to-text extraction via `scripts/extract_file.py` |
| Audio | `.mp3`, `.wav`, `.ogg`, `.m4a`, `.flac` | Transcribe via speech-to-text tools if available, or note as audio source |
| Archives | `.zip`, `.tar.gz`, `.rar` | List contents and process each file within |
| Binary/Doc | `.docx`, `.pptx`, `.odt` | Extract text via `scripts/extract_file.py` |

## Workflow

### 1. Locate the File

- If the user sends a file via Telegram, it's at `/home/fahmibakeri/.hermes/cache/documents/`
- If the user provides a path, use it directly
- Check `/home/fahmibakeri/.openclaw/media/inbound/` for recent PDFs
- For images from Telegram, check `/home/fahmibakeri/.hermes/cache/images/`

### 2. Extract Content (method depends on file type)

**For PDFs:**
```bash
python3 /home/fahmibakeri/.hermes/skills/create-markdown/scripts/extract_file.py <path>
```

**For images:** Use `vision_analyze` with a descriptive question like:
```
Analyze this image in detail. Describe all text, visual elements, layout, colors, charts, and any data presented.
```

**For text/code files:** Use `read_file` to get the full content.

**For CSV/tabular:** Use Python with pandas to parse and summarize:
```python
import pandas as pd
df = pd.read_csv(path)
print(f"Shape: {df.shape}")
print(f"Columns: {list(df.columns)}")
print(df.describe())
print(df.head(10).to_string())
```

**For HTML:** Use `scripts/extract_file.py <path>` which strips HTML tags and extracts readable text + metadata.

**For audio:** If transcription tools are available, use them; otherwise note the file as untranscribed audio.

**For DOCX/PPTX:** Use `scripts/extract_file.py <path>` which uses `python-docx` or `python-pptx` to extract text.

### 3. Analyze Content

Identify:
- Document/File type and purpose
- Key sections, themes, and structure
- Important data points, figures, or code snippets
- Relationships and connections
- Key takeaways

### 4. Generate Markdown Report

Format the report with the following structure:

```markdown
---
title: <Descriptive Title>
source: <original_filename.ext>
source_type: <pdf|image|code|text|csv|html|audio|...>
date: <YYYY-MM-DD>
tags: [hermes, analysis, <type>]
---

# <Descriptive Title>

## Overview
<2-3 sentence summary of what this file contains>

## Content Analysis

### <Section 1 — descriptive heading based on content>
<Detailed analysis, structured notes, or extracted content>

### <Section 2>
<More content>

## Key Takeaways
- <Bullet point 1>
- <Bullet point 2>
- <Bullet point 3>

## Raw Content / Excerpts
> <Notable excerpt or data>

---
*Generated from `<filename.ext>` on `<date>`*
```

**Report writing guidelines:**
- Be comprehensive — the report should be a standalone reference that captures the essence of the source
- Structure logically based on the source material's natural sections
- For code files: include a summary of what the code does, key functions/classes, and architectural notes
- For images: describe all visible elements, text, data, charts, and visual hierarchy
- For CSV/data: describe columns, data types, summary statistics, trends, and notable patterns
- Use tables where appropriate for structured data
- Include relevant code blocks with syntax highlighting for code files

### 5. Save to Obsidian Vault

- **Vault path:** `/home/fahmibakeri/famb vault`
- **Filename format:** `<ShortDescriptiveTitle>_hermes.md`
  - Use underscores for spaces, keep it concise but readable
  - Example: `UEDM_Volumetrics_Best_Practices_Apr_2026_hermes.md`
  - Example: `Python_DataPipeline_Analysis_hermes.md`
  - Example: `Architecture_Diagram_Review_hermes.md`
- **Suffix:** Always append `_hermes` before `.md` so the user can distinguish AI-generated notes from manual ones
- Write the full frontmatter + content to the file

### 6. Verify

Read the file back to confirm it was written correctly.

## Output Format Reference

See existing examples in the vault for format reference:
- `/home/fahmibakeri/famb vault/UEDM Volumetrics Best Practices_Apr 2026_hermes.md`

## Scripts

### `scripts/extract_file.py`

Universal file extraction script. Detects file type by extension and extracts readable text content.

```bash
python3 /home/fahmibakeri/.hermes/skills/create-markdown/scripts/extract_file.py <path_to_file>
```

**Supported formats:**
- PDF (pdfplumber → PyMuPDF + Tesseract OCR fallback)
- DOCX (python-docx)
- PPTX (python-pptx)
- HTML (BeautifulSoup)
- CSV/TSV (pandas or csv module)
- Plain text (any extension — reads as-is)
- Archives (lists contents)

**Output:** Extracted text to stdout; errors/diagnostics to stderr.

## Setup (One-Time)

Install Python dependencies for file extraction:

```bash
cd /home/fahmibakeri/.hermes/skills/create-markdown
uv venv .venv --python 3.12
uv pip install --python .venv/bin/python pdfplumber pillow pytesseract pymupdf python-docx python-pptx beautifulsoup4 lxml pandas openpyxl
```

> **Note:** The script auto-detects `.venv` alongside the skill directory. Tesseract OCR engine must be installed system-wide for PDF OCR fallback.

## Pitfalls

- **Image files cannot be read by `read_file`** — always use `vision_analyze` for images
- **Large PDFs (>50 pages):** extract text first, then analyze in chunks. Use `scripts/extract_file.py` to get text, then process it
- **Password-protected files:** Note to user that protected files cannot be processed
- **Corrupted files:** Try alternative extraction methods or notify the user
- **Audio files:** Do NOT try to `read_file` or `vision_analyze` audio — note that transcription requires a speech-to-text tool. Skip audio and explain to the user
- **Binary files (exe, dll, .so):** Cannot be analyzed for content — note as binary
