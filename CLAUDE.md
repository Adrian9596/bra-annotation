# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This repo contains two standalone browser tools and one sub-repository, all for intimate apparel product development at Crossian:

| File / Folder | Tool |
|---|---|
| `index.html` | **Sketch Line Tool** — annotate garment sketches with lines and arrows, export PDF/JSON |
| `sample_request_builder_prototype.html` | **Sample Request Builder** — generate bilingual (EN/CN) factory sample request text |
| `tech-pack-3597-offline/` | **Tech Pack App** — full offline tech pack for Style 3597 (separate git repo with its own CLAUDE.md) |

## Development

**No build system.** All tools are single-file HTML or static assets. Open in a browser directly:

```bash
open index.html
open sample_request_builder_prototype.html
open tech-pack-3597-offline/index.html
```

Changes take effect on browser refresh. No compilation, no npm install needed for the HTML tools (the root `package.json` / `package-lock.json` are legacy and not required).

## How to Use

### Sketch Line Tool
1. Open `index.html` in a browser.
2. Load a garment sketch via the file input or paste from clipboard (Ctrl/Cmd+V).
3. Select a tool (straight line, curved line) and pick color, arrow type, and solid/dashed style from the toolbar.
4. Click and drag on the canvas to draw annotations. Click an annotation to select it; press Delete to remove it.
5. Zoom with scroll wheel; pan by holding Space and dragging.
6. Export: Ctrl/Cmd+Shift+P → PDF, Ctrl/Cmd+Shift+E → JSON, Ctrl/Cmd+Shift+I → import JSON.
7. Annotations auto-save to localStorage and restore on next open.

### Sample Request Builder
1. Open `sample_request_builder_prototype.html` in a browser.
2. Choose **Action** (Make / Send / Prepare) and **Sample Type** from the dropdowns.
3. Toggle chips to select **Sizes**, **Versions** (Lace / Solid), and **Colors**.
4. Optionally check **Extra Instructions** (Tech Pack, BOM, Construction Details, Urgent).
5. Add a free-text **Custom Note** if needed.
6. Choose **Output Format**: Auto (sentence for simple, list for complex), Sentence, or List.
7. Click **Copy English**, **Copy Chinese**, or **Copy Both** — the bilingual factory request is now on your clipboard.
8. Page refresh resets all inputs (no persistence).

### Tech Pack App
1. Open `tech-pack-3597-offline/index.html` in a browser — no internet required.
2. See its own `CLAUDE.md` inside `tech-pack-3597-offline/` for full usage and architecture details.

---

## Sketch Line Tool (`index.html`)

Single self-contained file — all CSS and JS are inline. Key architecture:

- **State**: a `state` object holds `annotations[]`, active tool, zoom, pan, draw color, arrow type, draw style (solid/dashed), and undo/redo stacks.
- **Canvas rendering**: every interaction calls `redraw()`, which repaints all annotations onto a `<canvas>`.
- **Annotations**: each annotation is `{ type: 'straight'|'curved', start, end, style, color, arrowType, ... }`. Curved lines use a cubic bezier with a computed control point.
- **Image**: loaded via file input or clipboard paste, stored as an `Image` object in `state.bgImage`. The canvas scales it with pan/zoom.
- **Export**: PDF is built from scratch as a raw PDF byte stream (no library). JSON export/import serialises `state.annotations` only.
- **Auto-save**: annotations are saved to `localStorage` on every change and restored on load.
- **Keyboard shortcuts**: Ctrl/Cmd+Z (undo), Ctrl/Cmd+Y (redo), Delete (delete selected), Ctrl/Cmd+Shift+P (PDF), Ctrl/Cmd+Shift+E (export JSON), Ctrl/Cmd+Shift+I (import JSON).

## Sample Request Builder (`sample_request_builder_prototype.html`)

Single self-contained file. Generates bilingual factory sample request text from a fixed `TERM` dictionary. Key points:

- `TERM` object holds all EN↔CN translations for actions, sample types, sizes, versions, colors, and instructions.
- `render()` is called on every input change and calls either `buildSentence()` or `buildList()` based on `formatMode` and selection complexity.
- `safeText()` strips invisible Unicode characters before clipboard writes.
- No state persistence — page refresh resets all inputs.

## Tech Pack Sub-Repo (`tech-pack-3597-offline/`)

This is a separate git repository. See its own `CLAUDE.md` for full architecture details.

**Key constraint — do not modify the `#photos` section** in `tech-pack-3597-offline/index.html` or the photo rendering logic in `app.js`.

When removing UI elements that `renderRoundEditor()` in `app.js` depends on (e.g. `#roundSelect`, `#roundDateInput`, `#roundBaseSizeInput`), always add null guards — otherwise `renderTables()` crashes before photos render:
```js
if (roundSelect) roundSelect.innerHTML = ...
if ($("#roundDateInput")) $("#roundDateInput").value = ...
```

**Cache busting**: bump the `?v=` query string on the CSS/JS `<link>`/`<script>` tags in `tech-pack-3597-offline/index.html` whenever deploying so browsers pick up changes.
