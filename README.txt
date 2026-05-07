# Clean RXN+

A ChemDraw add-in panel for cleaning up multi-step reaction schemes. Drag-and-drop card interface, live SVG preview, and a one-click auto-layout — built to fix the awkward spacing, overlapping reagent labels, and crooked arrows that pile up when you draw complex reactions in ChemDraw.

![ChemDraw](https://img.shields.io/badge/ChemDraw-25.5+-1a9e45) ![License](https://img.shields.io/badge/license-MIT-blue) ![Status](https://img.shields.io/badge/status-beta-orange)

## What it does

When you select a reaction scheme in ChemDraw and open the panel, Clean RXN+ parses the CDXML, lays out the molecules, arrows, and reagent labels into clean columns, and writes the computed positions back to the canvas. You can intervene at any point: drag cards to reorder them, push reagent stacks above or below the arrow, set per-card vertical gaps, or hold a card aside while you tune everything else.

It works on multi-step schemes (reactants → step 1 → step 2 → ... → products), respects ChemDraw's reaction scheme metadata when present, and falls back to spatial inference when not.

## Features

- **Live SVG preview** of the current layout with zoom controls — what you see is exactly what gets written to ChemDraw
- **Drag-and-drop card board** organized by reaction zone (Reactants, Steps, Products) with separate stacks for above-arrow and below-arrow reagents
- **One-click Auto-Layout** (`Ctrl+Shift+X`) cleans the selection without opening the panel
- **H-Respect** per-card toggle that nudges a card vertically to avoid overlap with neighbors
- **Balance above/below** mode for distributing stacked cards evenly around the arrow centerline
- **Auto-insert `+`** between reactants and products
- **Holding Area** for parking cards you want to exclude from layout temporarily
- **Named presets** for saving spacing/mode combinations across sessions
- **Undo** (`Ctrl+Z`) up to 20 steps
- **Debug tab** with raw CDXML in/out, session snapshot, log, and a thumbnail diagnostics panel

## How it works

1. **Select in ChemDraw.** Highlight the reactants, products, arrows, and any above/below labels. The panel parses the CDXML automatically when the selection changes.
2. **Inspect the board.** Each molecule and label appears as a draggable card. Columns represent spatial zones; the green-bordered card in each column is the anchor. Drag cards to reorder, move them between columns, or drop them in the Holding Area.
3. **Tune the layout.** Use the horizontal spacing slider and stack gap to control spacing. Toggle H-Respect on individual cards to auto-nudge them clear of overlapping neighbors. Use the arrow-column tools (`↕ Shift Label`, `⟺ Widen`, `↺`) to fix reagent label placement.
4. **Apply to ChemDraw.** Click **Apply to ChemDraw ↩** to write the computed `dx/dy` offsets back to the canvas. Or use **⚡ Auto-Layout** (`Ctrl+Shift+X`) for a one-shot clean without ever opening the board.

## Key concepts

| Term | Meaning |
|---|---|
| **Center card** | The anchor molecule in each column (green left border). Other cards stack above/below it. Reassign with "Set as Center". |
| **H-Respect** | Per-card toggle. When on, the layout engine nudges the card vertically to clear overlapping cards in neighboring columns. Useful for wide reagent labels next to compact molecules. |
| **Balance above/below** | Distributes a column's full stack evenly above and below the arrow centerline instead of anchoring to a specific card. |
| **Holding Area** | A parking zone for cards excluded from layout. Held cards aren't moved when Apply is clicked. |
| **Presets** | Named snapshots of your spacing/mode settings, saved to local storage and recallable across sessions. |

## Installation

Clean RXN+ is a single-file HTML add-in. To install:

1. Download `index.html` from this repository.
2. In ChemDraw, open the add-in manager and point it at the file (or drop it in your add-ins folder per ChemDraw's docs).
3. Open the panel from the add-ins menu and select a reaction scheme on the canvas.

ChemDraw 25.5 or newer is required for the add-in API used here.

## Architecture

The whole tool is one HTML file with embedded CSS and JavaScript. Major pieces:

- **`parseState(cdxml)`** — extracts molecules, arrows, and text into the flat `cards{}` data model and infers column structure from spatial coordinates and `<scheme>` metadata
- **`RxnEngine.calculateLayout(...)`** — computes `dx/dy` offsets for each card based on the current column/arrow-column structure, horizontal spacing mode, and H-Respect constraints
- **`RxnEngine.drawSvg(...)`** — renders the live preview SVG from the layout
- **`applyLayoutToChemDraw(...)`** — re-serializes the input CDXML with the computed offsets applied and pushes it back via the ChemDraw API
- **Drag-and-drop** — pointer-based, operates directly on the column/arrow-column arrays

State is intentionally flat: everything lives in `cards{}`, `columns[]`, `arrowCols[]`, `held[]`, and `arrows[]`, rebuilt fresh on every parse.

## Debugging

The Debug tab exposes:

- Raw input CDXML (editable, with **Simulate Render** to test arbitrary CDXML)
- Output CDXML (with **Force Push** to send it to the canvas without going through Apply)
- Session snapshot JSON
- Live log
- Thumbnail diagnostics panel (probes every layer of the card thumb pipeline — useful when running inside the add-in's embedded webview, which has its own quirks)

The **Copy Full Debug Bundle** button packages the input CDXML, initial preview SVG, state snapshot, applied changes, and output CDXML into a single JSON blob, which is the easiest thing to attach to bug reports.

## Known quirks

- **Card thumbnails** use Clean RXN+'s own SVG renderer (the same one as the main preview) rather than ChemDraw's `getSVG()` output. The native ChemDraw SVG path produced visually-blank thumbnails inside the add-in webview for environment-specific reasons; the in-house renderer is plainer but reliably visible.
- **Partial-structure selections** are skipped — select complete molecules.
- **`SupersededBy` graphics** are remapped to their replacement arrow IDs during parse so step metadata still resolves correctly.

## Credits

Built by **Michael A. Leitch, PhD candidate**, Department of Chemistry, University of Pittsburgh. Clean RXN+ grew out of day-to-day work drawing and formatting complex multi-step reaction schemes in ChemDraw.

- LinkedIn: [linkedin.com/in/michaelaleitch](https://www.linkedin.com/in/michaelaleitch)
- Email: Leitch.A.Michael@gmail.com
