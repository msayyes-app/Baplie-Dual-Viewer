# BAPLIE Dual Viewer — Worklog

## Date: 2025-03-04
## File: `/home/z/my-project/download/baplie_viewer.html`

All 6 improvements have been implemented in the single HTML file.

---

## IMPROVEMENT 1: Show Ship Name from BAPLIE/MOVINS

### Changes made:
1. **Added `extractShipName(raw)` function** (after `is45ft()` function, ~line 636): Parses the TDT segment from raw EDIFACT text. Looks for the last composite element containing `::` and extracts the vessel name after the last `::`. Returns the name string or `null`.

2. **Added `shipName1` and `shipName2` to the global `S` state object** (~line 645): `var S={...,shipName1:null,shipName2:null};`

3. **Modified file load handlers** (~lines 2633-2635): When loading files (both via file input and drag-drop), now calls `extractShipName(t)` and stores the result in `S.shipName1` or `S.shipName2`.

4. **Added ship name display spans in HTML** (~lines 398-399, 409-410): Added `<span id="ship1">` and `<span id="ship2">` elements next to the "Plan 1" / "Plan 2" labels, styled with accent color and monospace font.

5. **Modified `updateStats(pn)`** (~line 876): Now also updates the ship name display element with `🚢` prefix if a ship name is available.

---

## IMPROVEMENT 2: Fix View 2 Stern Bay Alignment / Mouseover

### Root cause:
In `drawSingleBay()`, `hitTest()`, `hitTestPos()`, and `getTierY()`, the trio lookup only checked for odd bays (`if(bay%2===1)`) or only checked `lo` and `hi` members. For even bays (mid bay of a trio), the trio lookup was skipped, causing visual misalignment and hitTest failures.

### Changes made:
1. **`drawSingleBay()`** (~line 940-952): Replaced the `if(bay%2===1)` condition with an unconditional trio lookup that checks `lo`, `mid`, AND `hi`. Now adds tiers from all three bays of the trio regardless of whether the current bay is odd or even.

2. **`hitTest()`** (~line 1115): Changed the condition from `_tr3.lo===bay||_tr3.hi===bay` to `_tr3.lo===bay||_tr3.mid===bay||_tr3.hi===bay`, and added tier collection from `_tr3.mid` when present.

3. **`hitTestPos()`** (~line 1125): Same fix as hitTest — added `||_tr4.mid===bay` to the condition and added mid bay tier collection.

4. **`getTierY()`** (~line 1191): Added `||_tr5.mid===bayNum` to the condition and added `addT(_tr5.mid)` when mid is present.

---

## IMPROVEMENT 3: Create New View "Vista 5" (Tall Slots, Adjustable Trio)

### Changes made:
1. **Added "Vista 5" button** in `updateTB()` (~line 1512): `<button class="tbtn" onclick="swV(pn,'5b')">Vista 5</button>`

2. **Modified `compLayout()`** (~lines 874-921): 
   - When `view==='5b'`, sets `effectiveView='3b'` to reuse 3b rendering logic
   - After `grps` is computed, calculates `cellH5b` based on available viewport height divided by max tier count across all trios
   - Calculates `slotW5b` to fill available width per trio (3 bays × rows)
   - Recomputes `rO[]` and `gw` with the new slot width
   - Sets `cellH` to `cellH5b` when view is `5b`

3. **Layout return value** already includes `cellH` and `slotW` which are used by `drawSingleBay()`, `drawGroup()`, `hitTest()`, and `hitTestPos()`.

---

## IMPROVEMENT 4: Proportional Overdimension (OOG)

### Changes made:
**Replaced the OOG drawing section** in `drawCell()` (~lines 1067-1117):

- **Before**: Fixed triangle size `var ts=Math.min(w*0.15,h*0.4,7)` for all directions
- **After**: Proportional triangles based on overhang amount relative to 250cm reference (standard container width):
  - **TOP overflow**: `topRatio = dim.top / 250`, `topTs = topRatio * CH2` (proportional to cell height / adjacent tier)
  - **PORT (left) overflow**: `portRatio = dim.port / 250`, `portTs = portRatio * w` (proportional to cell width / adjacent slot)
  - **STARBOARD (right) overflow**: Same logic as port, extending to the right
  - **FWD overflow** (only when no port/stbd): Shows as upward triangle proportional to width
  - **Generic OOG** (no detailed dims): Falls back to small fixed triangle
  - **Dimension text**: Still shown when cell width >= 40px

All ratios are capped at 1.0 (max) so triangles never exceed one full adjacent slot dimension.

---

## IMPROVEMENT 5: Weight Summary Per Row

### Changes made:
1. **Added weight summary in `drawSingleBay()`** (~lines 1002-1037):
   - After drawing all cells, calculates weight totals per row for deck (tier ≥ 70) and hold (tier < 70)
   - Draws deck weights above the deck section in blue (`#4fc3f7`) at position `MT_OFF-26`
   - Draws hold weights below the hold section in orange (`#ff8a65`) at position `lf+8`
   - Format: `XX.Xt` (tonnes with 1 decimal)

2. **Added weight summary in `drawGroup()`** (~lines 1054-1089):
   - Same logic for 2b/1b views
   - Deck weight position: `MT_OFF-16` (slightly different since drawGroup uses smaller title)

---

## IMPROVEMENT 6: Summary Tables Button (Resumen)

### Changes made:
1. **Added "Resumen" button** in `updateTB()` (~line 1508): Button with chart-bar icon that calls `openSummary(pn)`

2. **Added summary modal HTML** (~lines 482-492): Modal with header (icon + title + close button) and body div, similar structure to existing DG modal

3. **Added CSS for summary modal** (~lines 227-239): Styles for `#summaryModal`, `#summaryModalBox`, `#summaryModalHeader`, `.sum-table`, `.sum-table th`, `.sum-table td`, `.sum-total`, `.sum-tab`, `.sum-tab.active`

4. **Added JavaScript functions** (~lines 2671-2741):
   - `SUM` global state object with `pn` (panel number) and `tab` (active tab)
   - `openSummary(pn)`: Opens modal and triggers render
   - `closeSummary()`: Closes modal
   - Click-outside-to-close handler
   - `renderSummary()`: Builds dynamic HTML with:
     - 8 tabs: POD, POL, Tipo, Bay, Row, DG, RF, OOG
     - Groups filtered containers by selected criteria
     - Shows table with columns: key, EQ, Full, MT, 20', 40', 45', Peso (t), Peso medio (t)
     - TOTAL row at bottom with aggregated values
     - Uses `passF_noTrt()` to apply current filters

---

## Notes
- All UI text is in Spanish as required
- The HTML file remains a single standalone file
- No external dependencies were added
- All existing functionality is preserved
