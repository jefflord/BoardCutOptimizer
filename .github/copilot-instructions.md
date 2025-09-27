# AI Coding Assistant Instructions

These instructions orient AI agents contributing to this project.

## Project Snapshot
Single-page, self-contained wood panel cutting layout optimizer (`wood-cutting-optimizer.html`). Pure HTML/CSS/vanilla JS—no build tooling, no dependencies. Provides UI for defining required panel sizes, a stock sheet size, kerf thickness, and auto-generates a cutting layout using a simple bottom-left packing heuristic with optional manual drag adjustments.

## Core Concepts & Data Structures
- Panels: Array `panels = [{ length, width, qty }, ...]` maintained in memory; rendered in the sidebar via `renderPanels()`.
- Stock sheets: Single active sheet `stockSheets[0] = { length, width, qty }` used for scaling and placement.
- Pieces: Expanded instances of panels (with quantity) collected into `allPieces` in `optimizeCutting()`, sorted by area desc.
- Placed rectangles: `placedRects = [{ x, y, length, width, piece }, ...]` built during optimization.
- Scaling logic: Dynamic scaling to fit sheet inside 700×500 viewport while preserving proportions.

## Layout Algorithm (Bottom-Left Fit w/ Rotation Attempt)
1. Expand panel definitions into discrete pieces.
2. Sort pieces by area descending.
3. For each piece, generate candidate positions: (0,0) + right/below corners of already placed rectangles (with kerf offsets).
4. Sort candidate positions by lowest y then lowest x.
5. First position that fits without overlapping (tests normal, then rotated) is accepted.
6. Rotation is implemented by swapping `length/width` directly on the `piece` object when fit found.

## Kerf Handling
- Kerf (`kerfThickness` input) is treated as spacing buffer between rectangles and when computing candidate positions.
- Overlap test uses inclusive kerf spacing: adjust only in `canPlacePiece()` logic. Changing kerf affects packing density and should trigger full re-run of `optimizeCutting()`.

## UI Interaction Patterns
- Panels are editable inline (`onchange` -> `updatePanel()`).
- Adding a panel clears input fields and defaults quantity to 1.
- Re-optimizing occurs explicitly via Optimize button or implicitly after stock change (sheet cleared then `optimizeCutting()` called).
- Pieces are draggable within bounds (HTML5 drag API). Dragging does NOT recompute statistics (only visual reposition). Potential enhancement: update stats after manual moves or track conflicts.

## Statistics
`updateStats()` calculates: used area, waste, efficiency, number of cuts (count of placed rectangles). Assumes single sheet currently (`usedSheets` hard-coded to 1).

## Conventions & Style
- No modules; all functions and state live in global scope—avoid introducing tooling unless project direction changes.
- Use small, purpose-named functions; keep DOM queries cached only if performance becomes an issue.
- Favor additive enhancements inside existing file until complexity justifies refactor.
- Numeric precision: Inputs accept `step="0.01"`; display keeps raw float values except stats rounding.

## Safe Extension Guidelines
When adding features, follow these patterns:
- New options: add input under Options section; read value inside `optimizeCutting()` or new function.
- Multi-sheet support: likely introduce loop over sheets, aggregate stats; ensure scaling per sheet or create per-sheet containers.
- Export/print: generate SVG/Canvas from `placedRects` using current scale.
- Persist state: serialize `panels`, `stockSheets`, kerf to `localStorage` on change; hydrate on load before calling `optimizeCutting()`.

## Potential Edge Cases (Current Behavior)
- Extremely large/small dimensions: scaling may shrink pieces to sub-pixel—consider min visual size.
- Panels that cannot fit individually (bigger than stock) are silently skipped (no alert) because `findBestPosition` returns null.
- Rotation mutates original panel instance dimensions for that specific generated piece only (acceptable since piece object is ephemeral).

## Quick Reference: Key Functions
- `renderPanels()` — Renders sidebar list of panel definitions.
- `optimizeCutting()` — Orchestrates layout creation and stat updates.
- `findBestPosition()` — Generates and tests candidate placements (rotation-aware).
- `canPlacePiece()` — Bounds & collision detection including kerf spacing.
- `createPieceElement()` — DOM element creation with scaled positioning.
- `updateCuttingBoard()` — Computes scaled board dimensions.
- `updateStats()` — Computes area usage metrics.

## Example Enhancement Tasks for AI
- Add warning badge beside panels that didn't place.
- Add toggle to disable rotation attempts.
- Support multiple stock sheets (iterate optimization per sheet until all pieces placed).
- Add CSV import/export for panel list.

## Do Not
- Introduce frameworks/bundlers without explicit request.
- Over-engineer (e.g., converting to classes/modules) unless scaling beyond single file.

## Working Agreement
Keep diffs minimal and localized. Preserve existing visual design unless feature requires change. Provide small incremental improvements plus optional follow-up suggestions.

---
If anything above seems ambiguous (e.g., multi-sheet strategy, rotation rules), ask for clarification before large refactors.