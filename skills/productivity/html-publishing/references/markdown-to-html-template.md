# Markdown-to-HTML Conversion Template

## When to Use

Use when the user says "generate HTML", "convert to HTML", "save as HTML", or "make a web page from this" after creating a markdown document. This produces a standalone styled HTML page for sharing/printing.

Do NOT use for interactive web apps (→ `live-api-dashboard` or `sketch` instead) or reports needing the site-specific visual language (→ Section 2 of `html-publishing` instead).

## Workflow

### 1. Read source markdown

```bash
read_file(path="/path/to/document.md")
```

### 2. Run conversion

```python
import markdown
import re

with open("/path/to/document.md") as f:
    md_content = f.read()

html_body = markdown.markdown(
    md_content,
    extensions=['tables', 'markdown.extensions.fenced_code', 'markdown.extensions.nl2br']
)

# Clean double <br> inside table cells
html_body = re.sub(r'<br>\s*</td>', '</td>', html_body)
```

### 3. Wrap in template

Use this template:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{TITLE}</title>
    <style>
        * { margin: 0; padding: 0; box-sizing: border-box; }
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
            background: #f4f6f9;
            color: #1a1a2e;
            line-height: 1.7;
            padding: 0;
        }
        .doc-header {
            background: linear-gradient(135deg, #0a1628 0%, #1a3a5c 100%);
            color: white;
            padding: 40px 0 30px;
            border-bottom: 4px solid #f5a623;
        }
        .doc-header .container {
            max-width: 960px;
            margin: 0 auto;
            padding: 0 30px;
        }
        .doc-header h1 {
            font-size: 28px;
            font-weight: 700;
            letter-spacing: -0.3px;
            margin-bottom: 8px;
        }
        .doc-header .subtitle {
            font-size: 14px;
            color: #94a3b8;
        }
        .container { max-width: 960px; margin: 0 auto; padding: 0 30px; }
        main { padding: 40px 0 60px; }
        h2 {
            font-size: 22px; font-weight: 700; color: #0a1628;
            margin-top: 48px; margin-bottom: 16px;
            padding-bottom: 8px; border-bottom: 2px solid #e2e8f0;
        }
        h2:first-of-type { margin-top: 0; }
        h3 { font-size: 18px; font-weight: 600; color: #1e3a5f; margin-top: 32px; margin-bottom: 12px; }
        p { margin-bottom: 16px; }
        blockquote {
            background: #f0f4ff;
            border-left: 4px solid #3b82f6;
            padding: 20px 24px;
            margin: 24px 0;
            border-radius: 0 8px 8px 0;
            font-size: 15px;
            color: #1e293b;
        }
        blockquote p { margin-bottom: 0; }
        hr { border: none; height: 1px; background: linear-gradient(to right, transparent, #cbd5e1, transparent); margin: 40px 0; }
        table {
            width: 100%; border-collapse: collapse; margin: 20px 0 28px;
            font-size: 14px; border-radius: 8px; overflow: hidden;
            box-shadow: 0 1px 3px rgba(0,0,0,0.08);
        }
        th, td { padding: 12px 16px; text-align: left; border-bottom: 1px solid #e2e8f0; }
        th { background: #1a3a5c; color: white; font-weight: 600; font-size: 13px; text-transform: uppercase; letter-spacing: 0.5px; }
        td { background: white; }
        tr:last-child td { border-bottom: none; }
        tr:hover td { background: #f8fafc; }
        td:first-child { font-weight: 600; color: #1a3a5c; }
        ul, ol { margin: 12px 0 20px 24px; }
        li { margin-bottom: 6px; }
        strong { color: #0a1628; }
        .doc-footer { background: #0a1628; color: #64748b; padding: 20px 0; font-size: 13px; text-align: center; }
        @media (max-width: 640px) {
            .doc-header h1 { font-size: 22px; }
            .container { padding: 0 16px; }
            table { font-size: 13px; }
            td, th { padding: 10px 12px; }
        }
        @media print {
            body { background: white; }
            .doc-header { background: #0a1628 !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
            th { background: #1a3a5c !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
            blockquote { background: #f0f4ff !important; -webkit-print-color-adjust: exact; print-color-adjust: exact; }
        }
    </style>
</head>
<body>
    <header class="doc-header">
        <div class="container">
            <h1>{DISPLAY_TITLE}</h1>
            <div class="subtitle">{SUBTITLE_METADATA}</div>
        </div>
    </header>
    <main class="container">{HTML_BODY}</main>
    <footer class="doc-footer">
        <div class="container"><em>{FOOTER_TEXT}</em></div>
    </footer>
</body>
</html>
```

### 4. Save

Save alongside the markdown file with `.html` extension (e.g. `My_Report_hermes.html`).

### 5. Dependencies

```bash
pip install markdown
```
