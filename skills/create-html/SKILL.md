---
name: create-html
description: >-
  Take any file input (PDF, image, text, code, CSV, HTML, audio, etc.),
  extract/analyze its content, and generate a beautiful standalone HTML report
  page. Saves locally, publishes to GitHub Pages (fahmiamni/github.io), and
  updates the index page with a card entry for the new file. Every generated
  page includes a backlink to index.html.
tags: [html, report, github-pages, analysis, pdf, image, code, csv, data-viz]
---

# Create HTML Reports from Any File

## Overview

Take **any file** the user provides → extract its content → generate a polished, standalone HTML report page → publish it on the `fahmiamni/github.io` GitHub Pages repo → add a card to `index.html`.

The HTML reports follow the existing site visual language: dark navy header, white card sections, accent-colored left borders, and clean typography — matching the style of existing pages like `iraq_2026_growth.html`, `petronas_top5_analysis.html`, and `DMR11_WellX_ZPEC.html`.

## Workflow

### 1. Locate the File

- If user sends via Telegram: `/home/fahmibakeri/.hermes/cache/documents/` or `/home/fahmibakeri/.hermes/cache/images/`
- If user provides a path: use it directly
- Also check `/home/fahmibakeri/.openclaw/media/inbound/` for recent PDFs

### 2. Extract Content (method depends on file type)

| Type | Extensions | Method |
|------|-----------|--------|
| PDF | .pdf | `scripts/extract_file.py` (pdfplumber → OCR) |
| Images | .png, .jpg, .webp, etc. | `vision_analyze` tool |
| Text/MD/Code | .txt, .md, .py, .js, .html, .json, etc. | `read_file` |
| CSV/XLSX | .csv, .tsv, .xlsx | Python + pandas |
| DOCX/PPTX | .docx, .pptx | `scripts/extract_file.py` |
| HTML | .html, .htm | `scripts/extract_file.py` |

> The universal extractor lives at `/home/fahmibakeri/.hermes/skills/create-markdown/scripts/extract_file.py` — reuse it here.

### 3. Design the HTML Page

Use the **site's existing visual language** — this is NOT a generic template. Match these design tokens:

**Color Palette (from index.html):**
```css
--dark:   #0f1b2d;
--blue:   #0032A0;
--light:  #e8edf5;
--mid:    #c5d0e6;
--green:  #1a7a4a;
--amber:  #d4820a;
--red:    #c0392b;
--purple: #6b21a8;
--text:   #1a1a2e;
--muted:  #5a6a7a;
--oc:     #ff6b35;
```

**Design pattern:**
- Dark navy header (`background: var(--dark)`) with a `4px solid var(--blue)` bottom border
- The report title in white, metadata in `var(--mid)`
- White body background (`var(--light)`)
- Content sections as white rounded cards with left accent borders
- Clean system font stack: `'Segoe UI', 'Calibri', sans-serif`
- Responsive (works on mobile) — use `max-width: 960px` centered container
- Footer with muted text on dark background

Always compose the HTML directly with `write_file` — do NOT use the `markdown-to-html` conversion script from the other skill.

### 4. Generate HTML Structure

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Report Title</title>
  <link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 100 100'><text y='.9em' font-size='90'>📄</text></svg>">
  <style>
    /* Site design tokens + your content styles */
    :root {
      --dark: #0f1b2d;
      --blue: #0032A0;
      --green: #1a7a4a;
      --amber: #d4820a;
      --red: #c0392b;
      --purple: #6b21a8;
      --light: #e8edf5;
      --mid: #c5d0e6;
      --text: #1a1a2e;
      --muted: #5a6a7a;
      --oc: #ff6b35;
    }
    /* ... full styling */
  </style>
</head>
<body>
  <!-- Header -->
  <header>
    <div class="header-inner">
      <a href="index.html" class="back-link">← Back to Index</a>
      <h1>Report Title</h1>
      <p class="meta">Source: filename.ext | YYYY-MM-DD</p>
    </div>
  </header>

  <!-- Main Content -->
  <main>
    <section class="card">
      <h2>Overview</h2>
      <p>...</p>
    </section>

    <section class="card">
      <h2>Sections...</h2>
      <!-- Tables, lists, code blocks, data visualizations -->
    </section>
  </main>

  <!-- Footer -->
  <footer>
    <p>Generated from <em>filename.ext</em> on YYYY-MM-DD</p>
    <p><a href="index.html">← Back to Index</a></p>
  </footer>
</body>
</html>
```

**Backlink requirement:** Every page MUST include a link back to `index.html` near the top (in the header) and optionally again in the footer. Use the existing site pattern: `<a href="index.html" class="back-link">← Back to Index</a>`.

### 5. Content Styling Guidelines

- Use **section cards** with white background, border-radius, and a left accent border (4px solid var(--blue) or var(--green)/var(--amber)/var(--purple) depending on content type)
- Tables should have dark navy headers (`background: #1a3a5c; color: white;`) and alternating white/gray rows
- Data values should be visually prominent (larger font, bold)
- Use `var(--muted)` for secondary text, context labels
- Charts/data visualizations: use inline SVG or HTML-only representations (horizontal bars, color-coded indicators)
- Code snippets in monospace with a subtle gray background
- Citations/quotes: indented with a left border in `var(--blue)`

### 6. Save the HTML File

Save to the GitHub Pages repo clone:

```
/home/fahmibakeri/.openclaw/workspace/github.io/<filename>.html
```

**Filename convention:**
- Use `snake_case.html` (underscores, no spaces)
- Keep the name descriptive but concise
- Example: `iraq_2026_growth.html`, `petronas_top5_analysis.html`, `DMR11_WellX_ZPEC.html`

### 7. Update index.html

Open `/home/fahmibakeri/.openclaw/workspace/github.io/index.html` and:

1. **Find the best category** for the new page. Existing categories:
   - Oil & Gas (icon: O&G)
   - Drilling & Reports (icon: DMR)
   - HR & Documents (icon: HR)
   - AI & GitHub Stats (icon: AI)

   If none fit, create a new category section following the same pattern:
   ```html
   <div class="category">
     <div class="cat-header">
       <span class="cat-icon">ICON</span>
       <span class="cat-title">Category Name</span>
       <span class="cat-count">1 report</span>
     </div>
     <ul class="link-list">
       <!-- new item goes here -->
     </ul>
   </div>
   ```

2. **Increment the category count** (e.g., "2 reports" → "3 reports")

3. **Add a new `<li>` entry** at the TOP of the list (most recent first) with:
   - `class="list-link"` — optionally add `accent-green`, `accent-amber`, `accent-red`, `accent-purple`, or `accent-oc` for colored left border
   - `href="filename.html"`
   - Title in `<span class="link-title">`
   - Description in `<span class="link-desc">`
   - Tag in `<span class="link-tag">`

   Follow the existing pattern exactly — read the file and match the structure.

4. Use `patch` to make targeted edits (not a full file rewrite).

### 8. Commit and Push

```bash
cd /home/fahmibakeri/.openclaw/workspace/github.io
git add <filename>.html
git add index.html
git commit -m "Add: <descriptive title> report"
git push
```

### 9. Verify Deployment

```bash
sleep 30
curl -s -o /dev/null -w "%{http_code}" "https://fahmiamni.github.io/<filename>.html"
```

Expected: `200`. The URL is `https://fahmiamni.github.io/<filename>.html`.

### 10. Deliver the URL

Send the published URL to the user.

## Report Content Guidelines

**For PDFs:** Extract text first, identify document type/purpose, structure into sections matching the source, include key data/tables/figures.

**For images:** Describe all visual elements, text, charts, layout, and data presented. Use image analysis from `vision_analyze`.

**For code files:** Summary of code purpose, key functions/classes, architecture, dependencies, and usage notes. Include syntax-highlighted code blocks.

**For CSV/data:** Summary statistics, column descriptions, trends, outliers, and visual representations (inline HTML bar charts, color-coded tables).

**For audio:** If transcription is possible, use it; otherwise note the file as untranscribed audio.

## Design References

See existing pages in the repo for visual reference:
- `file:///home/fahmibakeri/.openclaw/workspace/github.io/iraq_2026_growth.html`
- `file:///home/fahmibakeri/.openclaw/workspace/github.io/petronas_top5_analysis.html`
- `file:///home/fahmibakeri/.openclaw/workspace/github.io/DMR11_WellX_ZPEC.html`

## Pitfalls

- **Do NOT use `markdown-to-html` or `python markdown` library** — compose HTML directly to match the site's visual language
- **Always save to the repo clone** (`/home/fahmibakeri/.openclaw/workspace/github.io/`), not to Obsidian
- **Backlink is required** — every page must link back to `index.html`
- **Push may fail** if remote has advanced — use `git pull --rebase origin main` then retry
- **GitHub Pages delay** — deployment takes ~20-30 seconds; don't report 404 immediately
- **For images:** never use `read_file` — use `vision_analyze` instead
- **For audio:** never use `read_file` or `vision_analyze` — note that it's untranscribed
- **Card placement:** add new cards at the TOP of their category list (most recent first)
