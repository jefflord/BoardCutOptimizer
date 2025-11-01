# Wood Cutting Optimizer

A self-contained, browser-based tool for optimizing wood panel cutting layouts. Automatically generates efficient cutting plans from stock sheets and required panel dimensions using a bottom-left packing heuristic with optional manual adjustment.

**Live Tool:** Open `wood-cutting-optimizer.html` in any modern web browser—no installation or build process required.

---

## Features

### Panel & Stock Management
- **Define panels** with custom lengths, widths, quantities, and optional labels
- **Import panels** from SubBox data (structured text format) for quick batch entry
- **Multiple stock sheet types** with flexible quantities—pieces automatically spill to secondary sheets
- **Decimal and fractional display** (auto-converts to nearest 1/16″ for reference)

### Layout Optimization
- **Automatic layout** using bottom-left fit packing algorithm
- **Rotation support** — pieces are tested both orientations to find best fit
- **Kerf (blade thickness)** configurable to account for material loss during cutting
- **Multi-sheet support** — automatically spreads pieces across multiple stock sheet instances
- **Visual feedback** on unplaced pieces if they exceed available sheet space

### Visualization & Interaction
- **Interactive canvas** showing piece placement on cutting boards
- **Drag-to-reposition** individual pieces (visual only; does not recalculate layout)
- **Click to select** pieces or sheets for detailed statistics
- **Zoom controls** (25–300%) for detailed inspection or overview
- **Pan & scroll** for navigating large layouts

### Statistics & Analytics
- Global metrics: used sheets, total area used, waste area, cut count
- Per-sheet breakdown: dimensions, area efficiency, piece count
- Per-piece details: dimensions, position, label
- Unplaced piece warnings with highlight badges

### Export & Persistence
- **Save/load profiles** — preserve panels, sheet definitions, kerf, zoom, and export settings
- **Export sheets to PNG** with configurable resolution (2–40 px/inch)
- **Browser storage** — state automatically saves and restores on page reload

---

## Getting Started

### Basic Workflow

1. **Open the tool**: Load `wood-cutting-optimizer.html` in your browser
2. **Define panels**: Enter length, width, quantity, and optional label; click **Add Panel**
   - Or paste multiple panels via **Import / Paste Panels** using SubBox format
3. **Set stock sheet**: Adjust default 96″ × 48″ or add multiple sheet types with quantities
4. **Adjust options** (optional):
   - Set kerf thickness to account for blade width (default 0.25″)
   - Enable **Show fractions in diagram** to overlay 1/16″ measurements
5. **Optimize layout**: Click **Optimize Layout** button
6. **Review & adjust**:
   - Hover over pieces to see dimensions
   - Click a piece to view details in the panel details panel
   - Drag pieces to manually reposition (experimental; does not recalculate)
   - Click a sheet to view sheet-specific statistics
7. **Export** (optional): Click **Export PNG** after selecting a sheet

### Default Example

The tool loads with a sample project:
- 7 panel types (A–G) with various dimensions and quantities
- 96″ × 48″ stock sheet
- 0.25″ kerf thickness

Modify or clear the sidebar to test your own dimensions.

---

## Panel Import Format

Use the **Import / Paste Panels** feature to bulk-load panel definitions from structured text:

### SubBox Format
```
Detail             Size              Quantity
Top / Bottom       24.00" × 15.43"   2
Front              20.81" × 9.50"    1
Side               18.75" × 10.25"   2
```

**Requirements:**
- First row: headers (`Detail`, `Size`, `Quantity`)
- Dimensions format: `L × W` in decimal inches (quotes optional)
- Tab or multiple spaces separate columns
- One panel per row

**Parsing:**
1. Paste data into the textarea
2. Click **Parse** to validate
3. Review messages for any errors
4. Click **Add Parsed Panels** to append to your list

---

## Algorithm Overview

### Bottom-Left Fit with Rotation

The optimizer places pieces using a heuristic packing strategy:

1. **Expand definitions**: Convert panel records (length, width, qty) into individual piece instances
2. **Sort by area**: Larger pieces are placed first (descending area order)
3. **Generate candidates**: For each piece, test placement at:
   - Origin (0, 0)
   - Right edge of each already-placed rectangle (+ kerf offset)
   - Below each already-placed rectangle (+ kerf offset)
4. **Evaluate positions**: Sort candidates by lowest-y, then lowest-x
5. **Test fit**: For the best candidate position, test both normal and rotated (swapped length/width) orientations
6. **Place or skip**: If fit found, place piece; otherwise, leave unplaced and flag in sidebar

### Multi-Sheet Logic

- Stock sheet definitions are expanded (e.g., 2× 96″×48″ becomes two 96″×48″ instances)
- Optimizer assigns pieces sequentially to sheets
- If a piece doesn't fit on current sheet, it spills to the next available sheet
- Unplaced pieces remain flagged if all sheets are exhausted

### Kerf Handling

Kerf thickness is applied as a spacing buffer:
- Between pieces on the board
- When computing candidate positions (offset from existing edges)
- Kerf **does not** reduce the dimensions of the stock sheet itself

---

## Saving & Loading (Profiles)

### Save a Profile
1. Enter a profile name in the text input (e.g., "Kitchen Cabinet Kit")
2. Click **Save**
3. Current panels, stock sheets, kerf, fractions toggle, zoom level, and export resolution are stored

### Load a Profile
- Click a profile name in the **Profiles** list to restore its saved state

### Storage
- Profiles are stored in browser `localStorage`
- Each browser/device maintains its own profile library
- Clearing browser cache will delete profiles

---

## Options

### Kerf Thickness (Cut/Blade Thickness)
- **Default**: 0.25″
- **Purpose**: Accounts for material lost during cutting
- **Effect**: Increases spacing between pieces and reduces effective usable area

### Show Fractions in Diagram
- **Default**: Off
- **Purpose**: Display nearest 1/16″ fraction alongside each piece's decimal dimensions
- **Useful for**: Manual layout verification against imperial tape measures

---

## Statistics Panels

### Global Statistics
- **Used stock sheets**: Number of sheet instances after optimization
- **Total used area**: Sum of all placed piece areas (in²)
- **Total wasted area**: Unused area on all sheets (in²)
- **Total cuts**: Count of individual pieces placed
- **Unplaced pieces** (if any): Count of pieces that couldn't fit

### Sheet Details
- **Sheet dimensions** and area
- **Piece count** on sheet
- **Used / waste area** breakdown
- **Efficiency %**: (used area / total sheet area) × 100
- **Export PNG**: Generate a PNG image of the selected sheet at specified resolution

### Panel Details
- **Panel dimensions** (length × width)
- **Quantity** defined
- **Current placement**: x, y coordinates and rotation status

---

## Keyboard & Mouse Controls

| Action | Behavior |
|--------|----------|
| **Click piece** | Select and show details in panel details panel |
| **Click sheet** | Select and show sheet statistics & export button |
| **Drag piece** | Reposition piece visually (does not recalculate layout) |
| **Drag sidebar edge** | Resize sidebar (120–640px) |
| **Scroll zoom slider** | Zoom layout 25–300% |
| **Click Zoom Reset** | Return zoom to 100% |
| **Collapse sections** | Click section headers to expand/collapse sidebars (e.g., Panels, Stock Sheets) |

---

## Export to PNG

1. **Select a sheet** by clicking it on the canvas
2. **Click Export PNG** (or adjust resolution if desired)
3. **Adjust export resolution** (2–40 pixels/inch):
   - Lower: smaller file, faster processing
   - Higher: finer detail, larger file
4. PNG downloads automatically

**Note**: Export captures the sheet layout at current resolution settings, not the visible zoom level.

---

## Common Scenarios

### Scenario 1: Panel Doesn't Fit
- **Symptom**: Red "unplaced" badge appears next to panel in sidebar
- **Check**: Is the panel larger than the stock sheet? Try rotating the panel or reducing kerf.
- **Solution**: Add more stock sheets, or re-examine your panel dimensions.

### Scenario 2: Too Much Waste
- **Symptom**: Efficiency is very low (< 50%)
- **Try**: 
  - Increase kerf only if it's unrealistically low
  - Review panel ordering—reorder in sidebar (drag is not yet supported, but you can delete & re-add)
  - Consider alternative stock sheet size or orientation
  - Add complementary smaller panels to fill gaps

### Scenario 3: Comparing Multiple Designs
- **Use Profiles**: Save current setup (e.g., "Design A"), modify panels, **Optimize**, then save as "Design B"
- **Compare**: Click each profile to toggle between designs and review statistics

---

## Limitations & Known Issues

- **Rotation limitation**: Rotation is tested at placement time, but manual dragging does not update rotation or recalculate conflicts
- **Single optimization pass**: No iterative or simulated-annealing refinement; first-fit heuristic may not achieve global optimum
- **No multi-sheet balancing**: Pieces are placed sequentially; load is not balanced across sheets
- **Sub-pixel rendering**: Extremely small or large dimensions may render below visual threshold at low zoom
- **Browser-only**: No cloud sync; profiles are local to the browser/device

---

## Technology

- **HTML5**: Semantic structure
- **CSS3**: Flexbox layout, animations, responsive design
- **Vanilla JavaScript**: No frameworks or build tools required
- **Canvas API**: Used for PNG export via HTML5 Canvas
- **localStorage API**: Profile and state persistence

---

## Browser Compatibility

Tested and working on:
- Chrome/Chromium 80+
- Firefox 75+
- Safari 13+
- Edge 80+

Requires ES6 support (arrow functions, const/let, template literals).

---

## Future Enhancement Ideas

- **Drag-to-reorder panels** in sidebar
- **Custom rotation angles** (not just 90° flip)
- **Multiple packing algorithms** (genetic, simulated annealing)
- **Import/export from CAD formats** (DXF, SVG)
- **Cost calculator** ($ per sheet, material density)
- **Cut list generation** (detailed cutting instructions)
- **Undo/redo** for manual piece repositioning
- **Collision detection feedback** during drag
- **Dark mode** theme option
- **Print-optimized layout** (assembly diagrams)

---

## Contributing

This project is intentionally simple and self-contained (single HTML file, no dependencies). 

**Before major refactors:**
- Preserve vanilla JS approach unless explicitly changing project direction
- Keep all code in `wood-cutting-optimizer.html` unless splitting is necessary
- Follow existing naming conventions and comment style
- Test in at least two browsers

For small fixes or feature requests, open an issue or submit a pull request with a clear description.

---

## License

See LICENSE file (if applicable) or contact repository owner for usage terms.

---

## Support

- **Bug reports**: Open an issue with steps to reproduce
- **Feature requests**: Describe use case and expected behavior
- **Questions**: Check this README or existing issues first

---

**Happy cutting!** 🔨
