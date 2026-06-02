---
name: baoyu-infographic
description: "Infographics: 21 layouts x 21 styles (信息图, 可视化)."
version: 1.56.1
author: 宝玉 (JimLiu)
license: MIT
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [infographic, visual-summary, creative, image-generation]
    homepage: https://github.com/JimLiu/baoyu-skills#baoyu-infographic
---

# Infographic Generator

Adapted from [baoyu-infographic](https://github.com/JimLiu/baoyu-skills) for Hermes Agent's tool ecosystem.

Two dimensions: **layout** (information structure) × **style** (visual aesthetics). Freely combine any layout with any style.

## When to Use

Trigger this skill when the user asks to create an infographic, visual summary, information graphic, or uses terms like "信息图", "可视化", or "高密度信息大图". The user provides content (text, file path, URL, or topic) and optionally specifies layout, style, aspect ratio, or language.

## Options

| Option | Values |
|--------|--------|
| Layout | 21 options (see Layout Gallery), default: bento-grid |
| Style | 21 options (see Style Gallery), default: craft-handmade |
| Aspect | Named: landscape (16:9), portrait (9:16), square (1:1). Custom: any W:H ratio (e.g., 3:4, 4:3, 2.35:1) |
| Language | en, zh, ja, etc. |

## Layout Gallery

| Layout | Best For |
|--------|----------|
| `linear-progression` | Timelines, processes, tutorials |
| `binary-comparison` | A vs B, before-after, pros-cons |
| `comparison-matrix` | Multi-factor comparisons |
| `hierarchical-layers` | Pyramids, priority levels |
| `tree-branching` | Categories, taxonomies |
| `hub-spoke` | Central concept with related items |
| `structural-breakdown` | Exploded views, cross-sections |
| `bento-grid` | Multiple topics, overview (default) |
| `iceberg` | Surface vs hidden aspects |
| `bridge` | Problem-solution |
| `funnel` | Conversion, filtering |
| `isometric-map` | Spatial relationships |
| `dashboard` | Metrics, KPIs |
| `periodic-table` | Categorized collections |
| `comic-strip` | Narratives, sequences |
| `story-mountain` | Plot structure, tension arcs |
| `jigsaw` | Interconnected parts |
| `venn-diagram` | Overlapping concepts |
| `winding-roadmap` | Journey, milestones |
| `circular-flow` | Cycles, recurring processes |
| `dense-modules` | High-density modules, data-rich guides |

Full definitions: `references/layouts/<layout>.md`

## Style Gallery

| Style | Description |
|-------|-------------|
| `craft-handmade` | Hand-drawn, paper craft (default) |
| `claymation` | 3D clay figures, stop-motion |
| `kawaii` | Japanese cute, pastels |
| `storybook-watercolor` | Soft painted, whimsical |
| `chalkboard` | Chalk on black board |
| `cyberpunk-neon` | Neon glow, futuristic |
| `bold-graphic` | Comic style, halftone |
| `aged-academia` | Vintage science, sepia |
| `corporate-memphis` | Flat vector, vibrant |
| `technical-schematic` | Blueprint, engineering |
| `origami` | Folded paper, geometric |
| `pixel-art` | Retro 8-bit |
| `ui-wireframe` | Grayscale interface mockup |
| `subway-map` | Transit diagram |
| `ikea-manual` | Minimal line art |
| `knolling` | Organized flat-lay |
| `lego-brick` | Toy brick construction |
| `pop-laboratory` | Blueprint grid, coordinate markers, lab precision |
| `morandi-journal` | Hand-drawn doodle, warm Morandi tones |
| `retro-pop-grid` | 1970s retro pop art, Swiss grid, thick outlines |
| `hand-drawn-edu` | Macaron pastels, hand-drawn wobble, stick figures |

Full definitions: `references/styles/<style>.md`

## Recommended Combinations

| Content Type | Layout + Style |
|--------------|----------------|
| Timeline/History | `linear-progression` + `craft-handmade` |
| Step-by-step | `linear-progression` + `ikea-manual` |
| A vs B | `binary-comparison` + `corporate-memphis` |
| Hierarchy | `hierarchical-layers` + `craft-handmade` |
| Overlap | `venn-diagram` + `craft-handmade` |
| Conversion | `funnel` + `corporate-memphis` |
| Cycles | `circular-flow` + `craft-handmade` |
| Technical | `structural-breakdown` + `technical-schematic` |
| Metrics | `dashboard` + `corporate-memphis` |
| Educational | `bento-grid` + `chalkboard` |
| Journey | `winding-roadmap` + `storybook-watercolor` |
| Categories | `periodic-table` + `bold-graphic` |
| Product Guide | `dense-modules` + `morandi-journal` |
| Technical Guide | `dense-modules` + `pop-laboratory` |
| Trendy Guide | `dense-modules` + `retro-pop-grid` |
| Educational Diagram | `hub-spoke` + `hand-drawn-edu` |
| Process Tutorial | `linear-progression` + `hand-drawn-edu` |

Default: `bento-grid` + `craft-handmade`

## Keyword Shortcuts

When user input contains these keywords, **auto-select** the associated layout and offer associated styles as top recommendations in Step 3. Skip content-based layout inference for matched keywords.

If a shortcut has **Prompt Notes**, append them to the generated prompt (Step 5) as additional style instructions.

| User Keyword | Layout | Recommended Styles | Default Aspect | Prompt Notes |
|--------------|--------|--------------------|----------------|--------------|
| 高密度信息大图 / high-density-info | `dense-modules` | `morandi-journal`, `pop-laboratory`, `retro-pop-grid` | portrait | — |
| 信息图 / infographic | `bento-grid` | `craft-handmade` | landscape | Minimalist: clean canvas, ample whitespace, no complex background textures. Simple cartoon elements and icons only. |

## Output Structure

```
infographic/{topic-slug}/
├── source-{slug}.{ext}
├── analysis.md
├── structured-content.md
├── prompts/infographic.md
└── infographic.png
```

Slug: 2-4 words kebab-case from topic. Conflict: append `-YYYYMMDD-HHMMSS`.

## Core Principles

- Preserve source data faithfully — no summarization or rephrasing (but **strip any credentials, API keys, tokens, or secrets** before including in outputs)
- Define learning objectives before structuring content
- Structure for visual communication (headlines, labels, visual elements)

## Workflow

### Step 1: Analyze Content

**Load references**: Read `references/analysis-framework.md` from this skill.

1. Save source content (file path or paste → `source.md` using `write_file`)
   - **Backup rule**: If `source.md` exists, rename to `source-backup-YYYYMMDD-HHMMSS.md`
2. Analyze: topic, data type, complexity, tone, audience
3. Detect source language and user language
4. Extract design instructions from user input
5. Save analysis to `analysis.md`
   - **Backup rule**: If `analysis.md` exists, rename to `analysis-backup-YYYYMMDD-HHMMSS.md`

See `references/analysis-framework.md` for detailed format.

### Step 2: Generate Structured Content → `structured-content.md`

Transform content into infographic structure:
1. Title and learning objectives
2. Sections with: key concept, content (verbatim), visual element, text labels
3. Data points (all statistics/quotes copied exactly)
4. Design instructions from user

**Rules**: Markdown only. No new information. Preserve data faithfully. Strip any credentials or secrets from output.

See `references/structured-content-template.md` for detailed format.

### Step 3: Recommend Combinations

**3.1 Check Keyword Shortcuts first**: If user input matches a keyword from the **Keyword Shortcuts** table, auto-select the associated layout and prioritize associated styles as top recommendations. Skip content-based layout inference.

**3.2 Otherwise**, recommend 3-5 layout×style combinations based on:
- Data structure → matching layout
- Content tone → matching style
- Audience expectations
- User design instructions

### Step 4: Confirm Options

**Prefer `clarify` tool if available**: Use `clarify` to confirm options with the user. Since `clarify` handles one question at a time, ask the most important question first:

**Q1 — Combination**: Present 3+ layout×style combos with rationale. Ask user to pick one.

**Q2 — Aspect**: Ask for aspect ratio preference (landscape/portrait/square or custom W:H).

**Q3 — Language** (only if source ≠ user language): Ask which language the text content should use.

**Fallback when `clarify` tool is unavailable**: Present the options directly in your response as a numbered list. Ask the user to reply with their choice (e.g., "Pick a number: 1, 2, or 3"). Then proceed to Step 5 once they respond.

### Step 5: Generate Prompt → `prompts/infographic.md`

**Backup rule**: If `prompts/infographic.md` exists, rename to `prompts/infographic-backup-YYYYMMDD-HHMMSS.md`

**Load references**: Read the selected layout from `references/layouts/<layout>.md` and style from `references/styles/<style>.md`.

Combine:
1. Layout definition from `references/layouts/<layout>.md`
2. Style definition from `references/styles/<style>.md`
3. Base template from `references/base-prompt.md`
4. Structured content from Step 2
5. All text in confirmed language

**Aspect ratio resolution** for `{{ASPECT_RATIO}}`:
- Named presets → ratio string: landscape→`16:9`, portrait→`9:16`, square→`1:1`
- Custom W:H ratios → use as-is (e.g., `3:4`, `4:3`, `2.35:1`)

Save the assembled prompt to `prompts/infographic.md` using `write_file`.

### Step 6: Generate Image

**Primary approach** — Use the `image_generate` tool with the assembled prompt from Step 5.

- Map aspect ratio to image_generate's format: `16:9` → `landscape`, `9:16` → `portrait`, `1:1` → `square`
- For custom ratios, pick the closest named aspect
- On failure, auto-retry once
- Save the resulting image URL/path to the output directory

**Fallback when `image_generate` tool is unavailable** — Create the infographic as a self-contained HTML+CSS file and screenshot it via the browser:

1. **Render the layout+style in HTML/CSS**: Translate the selected layout's structure (e.g., horizontal flow for `linear-progression`, circle for `circular-flow`) and the selected style's aesthetic (colors, fonts, grid patterns) into a single `.html` file. Load the layout and style reference files first to get exact color codes, typography, and visual element descriptions.
2. **Embed all content**: Include the structured content from Step 2 — headline, section text, labels, stage descriptions, data points — directly in the HTML. Use `<div>`, flexbox/grid, and inline SVG for arrows and icons.
3. **Set viewport dimensions**: Match the aspect ratio (e.g., 1280×720 for landscape, 720×1280 for portrait, 800×800 for square).
4. **Open in browser**: Use `browser_navigate(url='file:///path/to/file.html')` to render it.
5. **Take screenshot**: The browser screenshot (`browser_vision` or the auto-captured screenshot at `/home/fahmibakeri/.hermes/cache/screenshots/`) produces the PNG. Copy it to the output directory as `infographic.png`.
6. **Share via MEDIA**: Use `MEDIA:/path/to/infographic.png` in your response to send it as an image to the user.

**HTML rendering tips by style**:
- `technical-schematic`: Dark blue background (#0D1B2A or #1E3A5F), CSS grid background, white text, amber accents (#F59E0B), monospace font (Share Tech Mono or similar), corner brackets as CSS `::before` pseudo-elements.
- `corporate-memphis`: Flat bright colors, sans-serif font, heavy borders, bold typography.
- `hand-drawn-edu`: Pastel backgrounds (#FEF3C7, #DBEAFE), rounded corners, soft shadows, wobbly-looking borders via border-radius.
- `craft-handmade`: Warm beige background, hand-drawn aesthetic, irregular borders, serif font.

### Step 7: Output Summary

Report: topic, layout, style, aspect, language, output path, files created.

## References

- `references/analysis-framework.md` — Analysis methodology
- `references/structured-content-template.md` — Content format
- `references/base-prompt.md` — Prompt template
- `references/layouts/<layout>.md` — 21 layout definitions
- `references/styles/<style>.md` — 21 style definitions

## Pitfalls

1. **Data integrity is paramount** — never summarize, paraphrase, or alter source statistics. "73% increase" must stay "73% increase", not "significant increase".
2. **Strip secrets** — always scan source content for API keys, tokens, or credentials before including in any output file.
3. **One message per section** — each infographic section should convey one clear concept. Overloading sections reduces readability.
4. **Style consistency** — the style definition from the references file must be applied consistently across the entire infographic. Don't mix styles.
5. **`image_generate` aspect ratios** — the tool only supports `landscape`, `portrait`, and `square`. Custom ratios like `3:4` should map to the nearest option (portrait in that case).

6. **Reading layout/style reference files** — The `skill_view` tool's `linked_files` dict only shows top-level references (`analysis-framework.md`, `structured-content-template.md`, `base-prompt.md`). Layout definitions at `references/layouts/<name>.md` and style definitions at `references/styles/<name>.md` are NOT listed there. To read them, use `read_file(path='<skill_dir>/references/layouts/<name>.md')` directly with the full filesystem path from `skill_dir` in the skill metadata. Alternatively, grep: `skill_view(name='baoyu-infographic', file_path='references/layouts/linear-progression.md')` often works despite the layout files not appearing in `linked_files`.

7. **`clarify` tool may be unavailable** — If the `clarify` tool isn't in your tool list, skip Step 4's clarify workflow. Instead present options inline in your response as a numbered list and ask the user to reply with their choice. Proceed once they respond.

8. **No source file to save** — The backup/comparison logic for `source.md` assumes the source comes as a file. If the source is conversation text (user's question + your prior explanation), skip the `source.md` backup step and go straight to analysis. The `source-{slug}.{ext}` file in the output structure is optional.
