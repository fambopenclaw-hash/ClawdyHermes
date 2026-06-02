---
name: html-publishing
description: >-
  Full pipeline for creating and publishing HTML content to GitHub Pages.
  Covers: markdown-to-HTML conversion, live API dashboards, HTML report
  generation from raw files (PDF, images, code, CSV), GitHub Pages deployment,
  repository verification, index page management, and verification.
  Aggregates create-html, publish-html-github-pages, markdown-to-html,
  and live-api-dashboard into a single class-level workflow.
version: 2.0.0
tags: [html, publishing, github-pages, dashboard, report, markdown, deployment]
metadata:
  hermes:
    tags: [html, publishing, github-pages, dashboard, report, markdown, deployment]
    related_skills: [website-content-analysis]
---

# HTML Publishing Pipeline

## Overview

This skill covers the full workflow of creating HTML content and publishing it to GitHub Pages. It consolidates four sub-workflows:

1. **Markdown → HTML** — Convert markdown documents to standalone styled HTML
2. **Report from raw files** — Generate rich HTML reports from PDFs, images, code, CSV, audio, etc.
3. **Live API dashboards** — Build auto-refreshing dashboards from public REST APIs
4. **GitHub Pages deployment** — Locate/clone repo, copy files, update index, commit, push, verify

The overall pipeline is:

```
Input (markdown / file / API) → Generate HTML → Locate GH Pages repo → Copy file →
Verify backlink → Update index.html → Commit & push → Verify deployment → Deliver URL
```

---

## 1. Markdown → HTML Conversion

Trigger on: "generate HTML", "convert to HTML", "save as HTML" after creating markdown.

### Technique

Use Python's `markdown` library with table/fenced-code extensions, wrapped in a responsive HTML template:

```python
import markdown
import re

with open("/path/to/document.md") as f:
    md_content = f.read()

html_body = markdown.markdown(
    md_content,
    extensions=['tables', 'markdown.extensions.fenced_code', 'markdown.extensions.nl2br']
)
html_body = re.sub(r'<br>\s*</td>', '</td>', html_body)
```

### Template

Use the responsive dark-header template with these design tokens:

- **Header**: `background: linear-gradient(135deg, #0a1628 0%, #1a3a5c 100%)` with `border-bottom: 4px solid #f5a623`
- **Body**: `background: #f4f6f9`, `color: #1a1a2e`, system font stack
- **Tables**: White cards with hover, dark navy headers, first-column bold
- **Blockquotes**: `border-left: 4px solid #3b82f6`, `background: #f0f4ff`
- **Print**: Preserve colors with `-webkit-print-color-adjust: exact`

See `references/markdown-to-html-template.md` for the complete HTML template.

### Save Convention

Save alongside the source markdown, same filename with `.html` extension. When also publishing, copy to the GH Pages repo.

Do NOT use this method for reports that need the site-specific visual language — use Section 2 instead.

---

## 2. HTML Report from Raw Files

Trigger on: any file input (PDF, image, code, CSV, audio) where the user wants a published HTML report.

### Input Processing by Type

| Type | Extensions | Method |
|------|-----------|--------|
| PDF | .pdf | pdfplumber → OCR |
| Images | .png, .jpg, .webp | `vision_analyze` tool |
| Text/MD/Code | .txt, .md, .py, .js, .html, .json | `read_file` |
| CSV/XLSX | .csv, .tsv, .xlsx | Python + pandas |
| DOCX/PPTX | .docx, .pptx | python-docx / python-pptx |
| HTML | .htm, .html | Parse and restyle |
| Audio | .mp3, .wav | Note as untranscribed |

### Site Design Language

Use these **site-specific design tokens** (matching `fahmiamni.github.io` visual language):

```css
:root {
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
}
```

**Design pattern**:
- Dark navy header with `4px solid var(--blue)` bottom border
- Title in white, metadata in `var(--mid)`
- White body background (`var(--light)`)
- Content sections as white rounded cards with left accent borders
- System font stack: `'Segoe UI', 'Calibri', sans-serif`
- Responsive: `max-width: 960px` centered container
- Footer with muted text on dark background

**Backlink requirement**: Every page MUST include `<a href="index.html" class="back-link">← Back to Index</a>` near the top.

**Content styling**:
- Section cards: white background, `border-radius`, `4px solid var(--blue)` left accent
- Tables: dark navy headers (`background: #1a3a5c; color: white;`), alternating rows
- Code: monospace with gray background
- Data viz: inline SVG or HTML-only (horizontal bars, color-coded indicators)

### Output Path

Save to the user's GitHub Pages repo clone (e.g. `/home/fahmibakeri/.openclaw/workspace/github.io/`). Filename: `snake_case.html`.

**⚠️ Always verify the remote URL before saving.** A directory named `github.io` may not be the actual Pages repo. Check with:
```bash
git -C /path/to/repo remote -v
```

---

## 3. Live API Dashboard Building

Trigger on: "live dashboard", "realtime stats", "auto-refreshing tracker", "live monitor".

### Architecture

Single self-contained HTML file that:
1. Fetches data from a public REST API via `fetch()` in the browser
2. Renders cards in a responsive grid
3. Auto-refreshes via `setInterval(fetchData, 60000)` (default 60s)
4. Animates number transitions with `requestAnimationFrame` (ease-out cubic)
5. Shows status indicator (live/error dot)
6. Includes summary bar with aggregate stats

### Structure

- **Header**: Back link, title, live badge, status indicator
- **Summary bar**: Total, top item, last-updated timestamp
- **Dashboard grid**: Responsive card grid (`repeat(auto-fill, minmax(260px, 1fr))`)
- **Each card**: Avatar icon, name, description, primary metric (animated), loading bar, secondary metric, rank badge, link
- **JavaScript**: `fetchAll()`, `updateUI()`, `animateCount()`, `buildSkeleton()`

### Example Data Sources

- GitHub API: `GET https://api.github.com/repos/{owner}/{repo}`
- CoinGecko: `GET https://api.coingecko.com/api/v3/simple/price`
- Open-Meteo: `GET https://api.open-meteo.com/v1/forecast`
- Any same-origin internal API

### Caching for Rate-Limited APIs

If the dashboard hits API rate limits (HTTP 403), use the **static API caching pattern**:
1. Add a GitHub Action that fetches data server-side on a cron schedule
2. Save as `data.json` in the repo
3. HTML fetches `data.json` instead of the live API
4. See `references/static-api-caching-via-gh-actions.md` for the full workflow template

### Pitfalls

- **GitHub API**: 60 req/hr unauthenticated. Add `Authorization` header with a token if needed.
- **CORS**: Not all APIs support browser CORS. Test with `curl` first.
- **Cold start**: Seed a placeholder `data.json` if using caching, so the page works before the first Action run.
- **Number animation**: Avoid refresh intervals < 10s to prevent animation jumps.

---

## 4. GitHub Pages Deployment

Trigger on: "publish to GitHub Pages", "put this on my site", "deploy".

### Step 1: Locate the Repo

**Check memory first**: The user may have saved the canonical clone path in memory. Check your memory for entries like "Clone at /path/to/repo" or "GitHub Pages repo."

Then search known locations:
```bash
# Check /tmp/ for freshly-cloned repos
ls /tmp/*github*/ 2>/dev/null
ls /tmp/*pages*/ 2>/dev/null

# Check home directory for legacy clones
find ~ -maxdepth 4 -type d \( -name "*.github.io" -o -name "github.io" \) 2>/dev/null

# Check if .openclaw/workspace/github.io exists (legacy location)
ls ~/.openclaw/workspace/github.io/.git 2>/dev/null && echo "FOUND"
```

**Verify remote** — do NOT trust directory names:
```bash
cd /path/to/repo
git remote -v
# Should show: origin git@github.com:OWNER/REPO.git matching the expected Pages repo
```

If not found, clone to `/tmp/{repo-name}/`:
```bash
git clone git@github.com:OWNER/REPO.git /tmp/{repo-name}/
```

The `gh` CLI is often unavailable. Prefer `git clone` with SSH as the primary method. If `gh` is available, you can also use `gh repo clone OWNER/REPO`.

### Step 2: Verify GitHub Pages is Enabled

```bash
curl -s "https://api.github.com/repos/OWNER/REPO" | python3 -c \
  "import sys,json; d=json.load(sys.stdin); print('has_pages:', d.get('has_pages'))"
```

If `has_pages: False`, enable it in Settings → Pages, or find the correct repo.

### Step 3: Check SSH Auth

```bash
ssh -T git@github.com 2>&1 | head -3
# Expected: "Hi username! You've successfully authenticated..."
```

### Step 4: Copy File and Add Backlink

```bash
cp "/path/to/source.html" /path/to/repo/
```

Ensure the HTML has a backlink to `index.html`. Match the existing site's pattern (search for `back-home` or `back-link` in existing files).

### Step 5: Update index.html

Read `index.html` to understand its structure (card-grid or link-list style).

**Card-grid style** — add a new card `<a>` at the TOP of the grid:
```html
<a class="card accent-{color}" href="filename.html">
  <span class="card-title">Title</span>
  <span class="card-desc">Description</span>
  <span class="card-tag">TAG</span>
</a>
```

**Link-list style** — add a new `<li>` at the TOP of the list:
```html
<li class="list-link accent-green" href="filename.html">
  <span class="link-title">Title</span>
  <span class="link-desc">Description</span>
  <span class="link-tag">TAG</span>
</li>
```

**If the page belongs in an existing category**: Find the matching `<div class="category">` block. Increment the category count (e.g., "2 reports" → "3 reports") and add the new `<li>` entry at the TOP of the `<ul class="link-list">` (newest first convention).

**If the page needs a new category**: The user might say "save under Daily Learning" or "add to a new section." Create a new `<div class="category">` block and insert it before the `<div class="about">` section (or at the top of the `.body` div). Follow the exact pattern of existing categories:

```html
<div class="category">
  <div class="cat-header">
    <span class="cat-icon">📖</span>
    <span class="cat-title">New Category Name</span>
    <span class="cat-count">1 report</span>
  </div>
  <ul class="link-list">
    <li>
      <a class="list-link accent-{color}" href="filename.html">
        <span class="link-main">
          <span class="link-title">Page Title</span>
          <span class="link-desc">Short description of the page content.</span>
        </span>
        <span class="link-tag">TAG</span>
      </a>
    </li>
  </ul>
</div>
```

Available accent classes: `accent-green` (O&G, DMR), `accent-amber` (daily learning), `accent-red`, `accent-purple` (HR), `accent-oc` (AI/trackers). No class = default blue accent.

Use `patch` for targeted edits to `index.html` — do NOT rewrite the entire file. After patching, re-read the relevant section to verify the structure is well-formed HTML (no unclosed tags).

### Step 6: Commit and Push

```bash
cd /path/to/repo
git add "filename.html"
git add index.html  # if updated
git commit -m "Add: {description} report"
git push
```

**On push rejection (remote has advanced):**
```bash
git pull --rebase origin main
git push
```

**On repo migration warning** (prints "This repository moved"):
```bash
git remote set-url origin git@github.com:NEW_OWNER/NEW_REPO.git
```
Do NOT ignore — the next push will fail. Also update memory with the new remote.

### Step 7: Verify Deployment

GitHub Pages takes ~20-30 seconds. Verify:

```bash
sleep 30
curl -s -o /dev/null -w "%{http_code}" "https://{username}.github.io/{repo-name}/{filename}"
# Expected: 200
```

**CDN cache**: Even with HTTP 200, the browser may serve stale DOM for 1-3 minutes.
- Verify raw content: `curl -s "https://..." | grep "unique-change-string"`
- Browser hard refresh (`Ctrl+Shift+R`) or `?nocache=1` query param
- Wait up to 2-3 minutes for CDN propagation

### URL Format

| Repo | Published URL |
|------|--------------|
| `user/user.github.io` | `https://user.github.io/file.html` |
| `user/reports` | `https://user.github.io/reports/file.html` |
| `user/site` | `https://user.github.io/site/file.html` |

If 404 persists, try alternate URL patterns and check if Pages is actually enabled via the API.

### Pitfalls

- **Wrong repo**: A directory named `github.io` might be `fahmiamni/reports` migrated from `fahmiamni/github.io` — verify the remote URL before pushing
- **No `gh` CLI**: Plain `git` + SSH works fine — no tool dependency
- **Spaces in filenames**: Use quotes or copy to `/tmp/` first

---

## 5. Workflow Decision Tree

```
User gives input
│
├─ Markdown file → Section 1 (Markdown → HTML) → Section 4 (Deploy)
├─ Raw file (PDF/image/code/CSV) → Section 2 (Report from file) → Section 4 (Deploy)
├─ API/tracker request → Section 3 (Live dashboard) → Section 4 (Deploy)
├─ Existing HTML → Section 4 (Deploy only)
└─ Mixed workflow: Section 1 → Section 4 (deploy without index update) → ... then index update via Section 2 patterns
```

## References

See the `references/` directory for:
- `references/markdown-to-html-template.md` — Full HTML template for markdown conversion (see Section 1)
- `references/live-dashboard-template.md` — Full dashboard HTML with JS (see Section 3)
- `references/repo-migration-handling.md` — Handling GitHub Pages repo renames
- `references/static-api-caching-via-gh-actions.md` — GitHub Actions caching pattern for rate-limited APIs
