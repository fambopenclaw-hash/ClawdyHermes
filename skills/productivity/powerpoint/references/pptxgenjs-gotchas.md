# PptxGenJS Setup & Table Gotchas

Session-specific learnings and workarounds.

## Globally Installed Modules

`npm install -g pptxgenjs` installs to the global npm root (`$(npm root -g)`), but Node's `require()` doesn't search that path by default.

**Fix:** Run scripts with:

```bash
NODE_PATH=$(npm root -g) node script.js
```

Or set it once per shell:

```bash
export NODE_PATH=$(npm root -g)
```

## Multi-Line Table Cells

Use `\n` inside a cell's text string to create line breaks within a single cell. No need for `breakLine: true` — plain `\n` is interpreted as a newline by PowerPoint.

```javascript
// ✅ Multi-line cell content
slide.addTable([
  ["Header"],
  ["Line 1\nLine 2\nLine 3"]
], { x: 1, y: 1, w: 8, border: { pt: 0.5, color: "C4CDD5" } });
```

## Factory Functions for Cell Options (Avoiding Mutation)

PptxGenJS mutates option objects in-place (Common Pitfall #7 in the main ref). For tables with many cells, use factory functions that return fresh objects each time rather than spreading a shared reference:

```javascript
// ✅ SAFE: Factory returns fresh object each call
function dataCell(text, isEven) {
  return {
    text: text,
    options: {
      fill: { color: isEven ? "EDF2F7" : "FFFFFF" },
      color: "2D3748",
      fontSize: 8,
      fontFace: "Calibri",
      valign: "top",
      margin: 2
    }
  };
}

// ❌ UNSAFE: Shallow spread still shares nested objects (fill, shadow, etc.)
const baseOpts = { fill: { color: "FFFFFF" }, color: "333333" };
let cell1 = { text: "A", options: { ...baseOpts } };
let cell2 = { text: "B", options: { ...baseOpts } };
// baseOpts.fill is still shared — PptxGenJS may mutate it
```

## Column Width Calculation for Wide Tables

For `LAYOUT_WIDE` (13.33" × 7.5"), leave 0.5" margins on each side = 12.33" usable. For tables with 7+ columns:

```javascript
let cw = [0.35, 1.5, 0.7, 1.9, 2.1, 3.3, 1.1]; // sum ≈ 10.95"
let totalW = cw.reduce((a, b) => a + b, 0); // verify fits within 12.33"
```

## Content Verification (No LibreOffice)

If `soffice` isn't available for image-based visual QA, verify content via python-pptx:

```bash
pip install python-pptx
python3 -c "
from pptx import Presentation
prs = Presentation('output.pptx')
for slide in prs.slides:
    for shape in slide.shapes:
        if shape.has_table:
            table = shape.table
            print(f'Table: {len(table.rows)} rows x {len(table.columns)} cols')
            for r_idx, row in enumerate(table.rows):
                for c_idx, cell in enumerate(row.cells):
                    print(f'  [{r_idx},{c_idx}] {cell.text[:80]}')
"
```
