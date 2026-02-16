---
name: presentations
description: Create professional PowerPoint presentations with slides, charts, tables, and images using PptxGenJS. Use when the user asks you to make a presentation, deck, or PPTX.
allowed-tools: Bash(node:*),Write,Read
---

# Creating PowerPoint Presentations

Use `pptxgenjs` (available at `/app/node_modules/pptxgenjs`) to create .pptx files.

## Quick start

Write a Node.js script and run it:

```bash
node /workspace/group/make-pres.mjs
```

## Script template

```javascript
import PptxGenJS from '/app/node_modules/pptxgenjs/dist/pptxgen.es.js';
const pres = new PptxGenJS();
pres.layout = 'LAYOUT_16x9';

// --- Color scheme (adjust per brand) ---
const C = {
  primary: '2E5090', secondary: '0088CC', accent: 'FF6B35',
  bg: 'FFFFFF', bgAlt: 'F5F5F5', text: '333333', textLight: '666666',
};

// --- Helper: header bar on content slides ---
function addHeader(slide, title) {
  slide.addShape(pres.ShapeType.rect, { x: 0, y: 0, w: '100%', h: 0.75, fill: { color: C.primary } });
  slide.addText(title, { x: 0.5, y: 0.12, w: 9, h: 0.5, fontSize: 28, bold: true, color: 'FFFFFF' });
}

// ========== SLIDES ==========

// 1. Title slide
const s1 = pres.addSlide();
s1.background = { color: C.primary };
s1.addText('Presentation Title', { x: 0.5, y: 1.5, w: 9, h: 1.2, fontSize: 48, bold: true, color: 'FFFFFF', align: 'center' });
s1.addText('Subtitle or date', { x: 0.5, y: 2.8, w: 9, h: 0.6, fontSize: 24, color: 'CCCCCC', align: 'center' });

// 2. Content slide with bullets
const s2 = pres.addSlide();
addHeader(s2, 'Key Points');
s2.addText([
  { text: 'First key point with detail', options: { bullet: true, fontSize: 20 } },
  { text: 'Second key point', options: { bullet: true, fontSize: 20 } },
  { text: 'Third key point', options: { bullet: true, fontSize: 20 } },
], { x: 0.7, y: 1.1, w: 8.6, h: 4, color: C.text, lineSpacingMultiple: 1.5 });

// Save
await pres.writeFile({ fileName: '/workspace/group/presentation.pptx' });
console.log('Done: /workspace/group/presentation.pptx');
```

## Slide layouts

### Title slide
```javascript
slide.background = { color: C.primary };
slide.addText('Title', { x: 0.5, y: 1.5, w: 9, h: 1.2, fontSize: 48, bold: true, color: 'FFFFFF', align: 'center' });
slide.addText('Subtitle', { x: 0.5, y: 2.8, w: 9, h: 0.6, fontSize: 24, color: 'CCCCCC', align: 'center' });
```

### Two-column (text + image)
```javascript
addHeader(slide, 'Analysis');
slide.addText('Key insight text\n\n• Metric up 25%\n• Cost down 15%', { x: 0.5, y: 1.0, w: 4.5, h: 4, fontSize: 18, color: C.text, valign: 'top' });
slide.addImage({ path: '/workspace/group/chart.png', x: 5.2, y: 1.0, w: 4.3, h: 4 });
```

### Chart slide
```javascript
addHeader(slide, 'Revenue Trend');
slide.addChart(pres.ChartType.bar, [
  { name: 'Q1', labels: ['Jan','Feb','Mar'], values: [45, 58, 72] },
  { name: 'Q2', labels: ['Jan','Feb','Mar'], values: [32, 45, 55] },
], { x: 0.5, y: 1.0, w: 9, h: 4, chartColors: [C.secondary, C.accent], showLegend: true, barDir: 'col' });
```

### Table slide
```javascript
addHeader(slide, 'Metrics');
slide.addTable([
  [
    { text: 'Metric', options: { bold: true, color: 'FFFFFF', fill: { color: C.primary } } },
    { text: 'Value', options: { bold: true, color: 'FFFFFF', fill: { color: C.primary } } },
    { text: 'Change', options: { bold: true, color: 'FFFFFF', fill: { color: C.primary } } },
  ],
  ['Revenue', '$2.4M', '+12%'],
  ['Users', '45,000', '+28%'],
], { x: 0.5, y: 1.0, w: 9, h: 2.5, rowH: 0.5, border: { pt: 1, color: 'CCCCCC' }, fontSize: 16 });
```

## Chart types

- `pres.ChartType.bar` — bar/column charts (use `barDir: 'col'` for vertical)
- `pres.ChartType.line` — line charts (add `lineSmooth: true` for curves)
- `pres.ChartType.pie` — pie charts (add `showPercent: true`)
- `pres.ChartType.doughnut` — donut charts
- `pres.ChartType.area` — area charts
- `pres.ChartType.scatter` — scatter plots

## Images

```javascript
// From file
slide.addImage({ path: '/workspace/group/image.png', x: 0.5, y: 1, w: 4, h: 3 });

// Generated diagram (mermaid or graphviz)
// First: mermaid -i diagram.mmd -o /workspace/group/diagram.png -b white -s 3
slide.addImage({ path: '/workspace/group/diagram.png', x: 0.5, y: 1, w: 9, h: 4 });
```

## Design rules

- One idea per slide. Less text, more visuals.
- Max 5-6 bullet points per slide, each under 2 lines.
- Title: 28-48pt bold. Body: 16-20pt. Never below 14pt.
- Use 2-3 colors max plus neutrals.
- 16:9 aspect ratio (10" x 5.625").
- Charts > tables > bullet points (prefer visual data).
- Add whitespace — don't fill every inch.
- Consistent header bar on all content slides.

## Converting existing PPTX to high-quality images

```bash
# PPTX → PDF → high-res PNGs (300 DPI)
libreoffice --headless --convert-to pdf presentation.pptx --outdir /workspace/group/
mkdir -p /workspace/group/slides
pdftoppm -png -r 300 /workspace/group/presentation.pdf /workspace/group/slides/slide
```

## Sending the PPTX

Email: attach via `mcp__gmail__send_email` with the file path.
Chat: convert to images first (see above), then send each via `mcp__nanoclaw__send_image`.
