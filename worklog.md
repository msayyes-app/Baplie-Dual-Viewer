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

---

## Date: 2026-05-08
## Session 2 Bug Fixes and Enhancements

### Issues Fixed:
1. **BAPLIE loading incomplete**: Changed `readFile()` to try UTF-8 first, then fallback to latin1. Added console.log of container count after parsing.
2. **Missing total container count**: Updated `updateStats()` to show `N CTNR · X FLL · Y MT · ZZZ,zt` format with explicit MT count.
3. **Removed summaries feature**: User said "Elimina los resúmenes de momento, no es eso lo que quiero" — the summary/Resumen tables feature was removed.

### New Features:
4. **Weight row sum font size increased**: Changed from 7px to 9px base (bold 700) for better readability. Adjustable via font size buttons.
5. **Rounding button**: Added "Redondeo" button that toggles `GF.wtRound` to round all weight values to integers.
6. **Font size +/- buttons**: Added `F+` and `F-` buttons that adjust `GF.wtFontSize` offset for all weight displays (containers + row summaries).
7. **POD color customization**: Added palette button and modal with color picker per POD. Users can now change any POD color via `setPodColor()`.
8. **Weight sum colors fixed for light mode**: Deck weights now use `#4dd0e1` (dark) / `#00838f` (light) — high contrast teal. Hold weights now use `#ffb74d` (dark) / `#bf360c` (light) — high contrast orange/red.
9. **Weight sum positioning**: Deck weights at `MT_OFF-5` (above tier numbers, below bay name with clear gap). Hold weights at `lf+9`.

### Code Changes:
- `readFile()`: Now tries UTF-8 first, falls back to latin1 on U+FFFD detection
- `GF` state: Added `wtRound: false` and `wtFontSize: 0`
- `fmtW()`: Supports rounding via `GF.wtRound`
- `drawCell()`: All font sizes include `+GF.wtFontSize` with `Math.max(5,...)` floor
- `drawSingleBay()` and `drawGroup()`: Weight summaries use 9+fontSize, bold 700, new colors, rounding support
- `updateStats()`: Shows FLL count, MT count, and total weight
- `resetAllFilters()`: Resets `wtRound` and `wtFontSize`
- Added HTML: POD color modal, Redondeo/Font+/Font- buttons, POD palette button
- Added JS: `toggleWtRound()`, `changeWtFont()`, `openPodColorModal()`, `closePodColorModal()`, `setPodColor()`

### Bug Fix (critical):
- **CRASH BUG**: `drawCell()` OOG code referenced `CH2` (a local variable from `drawSingleBay`/`drawGroup`) which was NOT in scope, causing `ReferenceError` that stopped rendering midway through the BAPLIE when OOG containers were encountered. Fixed by replacing `CH2` with `h` (the cell height parameter passed to drawCell).
- Reverted `readFile()` back to latin1-only encoding (same as original) since the UTF-8-first approach didn't help and the original was known to work.
---
Task ID: 1
Agent: main
Task: Implement user-requested changes to BAPLIE Dual Viewer

Work Log:
- Read current baplie_viewer.html (3097 lines) to understand view system, summary, logo, and menu
- Redesigned View 4 ('1b'): changed effectiveView from standalone to '2b' (like View 3), added tall slot computation to fill viewport, removed fullscreen behavior, removed zoom reset on view switch
- Updated swV() to simply set view and render without fullscreen/zoom reset
- Updated bay name click handler to switch to View 4 without fullscreen
- Updated drawGroup() with adaptive font sizes for tall cell mode (isBig flag)
- Updated drawCell() to use actual cell height (h) instead of global CH for font sizing, added tall cell threshold (h>=CH*2) for multi-line display mode
- Updated PageUp/PageDown handler for View 4 to not depend on fullscreen state
- Implemented summary multi-select: sumSelect() now accepts event parameter, CTRL+click adds to existing selection, normal click replaces selection
- Updated sum-count cell onclick to pass event: sumSelect(i,event)
- Changed logo MSaYYeS to blue gradient (#1565c0 → #42a5f5)
- Added .logo-title, .logo-red, .logo-norm CSS classes
- Changed BAPLIE Dual Viewer title: B, D, V letters in dark red gradient (#7f1d1d → #b91c1c)
- Changed menu default position to left sidebar (MENU_POS='left', body class 'menu-left')
- Added initialization IIFE to move filterBar into mainArea on page load

Stage Summary:
- View 4 now renders like View 3 (single-bay per group) with taller slots that fill the viewport
- No fullscreen toggle or zoom reset when switching views
- Summary supports CTRL+click for cumulative multi-selection
- Logo MSaYYeS in blue gradient, B/D/V in dark red gradient
- Menu defaults to left sidebar on page load

---
Task ID: 2
Agent: main
Task: Implement user's three new changes to BAPLIE Dual Viewer

Work Log:
- Vista 2 (5b): Added _drawCellFullInfo flag, set true only for view==='1b' (Vista 4), so Vista 2 shows only weight in tall cells
- Summary highlight: Added .sum-sel CSS class with yellow background, checked in renderSummary() if group's IDs are all in S.sel, added sumSyncIfOpen() calls to plan click handler and listado row click handler
- No-sel button: Modified drawCell() logic to dim containers not in current selection when No-sel is active and selection exists. Selected containers that don't pass strict filters still show normally. Updated tooltip text.

Stage Summary:
- Vista 2 now shows only weight even in tall slot mode (multi-line SZTP+POL reserved for Vista 4)
- Summary rows highlight yellow when their containers are selected
- No-sel button dims non-selected containers when selection exists (from summary, list, or plan)
---
Task ID: 1
Agent: Main Agent
Task: Add "Añadir R/S" button and R/S filter to BAPLIE Dual Viewer

Work Log:
- Added "Añadir R/S" button next to "Mover" in the header bar with orange styling
- Created RS Modal with paste textarea, live preview, and apply/cancel buttons
- Added "Limpiar" button in modal to clear all manual RS markings
- Implemented `parseRsIds()` to extract container IDs from pasted text
- Implemented `previewRsList()` for live validation of pasted container IDs
- Implemented `applyRsList()` to mark containers as RS (isRemocion=true)
- Added `_manualRsIds` Set to track manually-marked RS containers
- Added badge on "Añadir R/S" button showing count of manual RS marks
- Added `clearManualRs()` to remove all manual RS markings
- Added "R/S" filter button next to "POL=ESVLC" in filter bar
- Implemented accumulative filter: POL=ESVLC + R/S shows both types (OR logic)
- Updated `passF_noTrt()` with accumulative POL+RS filter logic
- Updated View 4 full-info display to show POD instead of POL for RS containers
- Updated RS circle rendering: in View 4 draws orange dot in corner, in other views draws full circle
- Added re-application of manual RS markings when new file is loaded
- Added cleanup of RS markings when panel is cleared
- Updated `resetAllFilters()` to reset rsFilter and button state
- Updated help modal with documentation for Añadir R/S and R/S filter

Stage Summary:
- "Añadir R/S" button allows pasting container IDs to mark as remociones
- RS containers show POD + removal circle in the plan view
- "R/S" filter button is accumulative with "POL=ESVLC" (shows both when both active)
- Manual RS markings persist across file reloads and are cleaned up on panel clear
- All changes saved to /home/z/my-project/download/baplie_viewer.html

---
Task ID: 2
Agent: Main Agent
Task: RS prevails over TRT + Add Secuencia (Bay Seq) feature

Work Log:
- Fixed TRT dimming so RS containers are not dimmed when both RS filter and TRT are active
- Added "Secuencia" button next to "Añadir R/S" with blue styling
- Created Secuencia Modal with paste textarea for 2-column Excel data (ID + Bay Seq)
- Added live preview with validation of pasted container IDs and Bay Seq numbers
- Implemented parseSeqLines() to extract container ID + sequence from pasted text
- Implemented applySeqList() to assign _baySeq property to containers
- Added _seqMap object to track manually-assigned sequences
- Added badge on "Secuencia" button showing count of assigned sequences
- Added "Limpiar" button in modal to clear all manual sequences
- Added "Seq" toggle button next to "Redondeo" in filter bar
- Modified fmtW() to return Bay Seq number when GF.seqShow is active
- Added W key handler to toggle between weight and Bay Seq display
- Added toggleSeqShow() function with wt-on styling
- Added Bay Seq display in tooltip (hover info)
- Added Bay Seq display in DG modal info
- Re-apply seq data when new file is loaded
- Clean up seq data when panel is cleared
- Updated resetAllFilters to reset seqShow and Seq button
- Updated help modal with Secuencia documentation and W shortcut

Stage Summary:
- RS containers now prevail over TRT dimming (not greyed out when both active)
- "Secuencia" button allows pasting container IDs + Bay Seq from Excel
- Bay Seq replaces weight display when Seq toggle or W key is active
- All changes saved to /home/z/my-project/download/baplie_viewer.html
---
Task ID: 1
Agent: main
Task: Enhance diff log panel with POD color mode, verification ticks, same/different hatch backgrounds, and inline POD color picker

Work Log:
- Added CSS styles for new diff log elements: .diff-same-hatch, .diff-diff-hatch, .diff-pod-dot, .diff-tick, .verified-row, .diff-color-pick-inline
- Added JavaScript variables: DIFF_POD_COLOR_MODE, DIFF_VERIFIED, _diffInlineColorPicker
- Added helper functions: hatchOf(bay), sameHatch(posA, posB) for determining hatch membership
- Rewrote refreshDiffList() to include POD color dots, verification ticks, same/different hatch backgrounds, and POD color mode
- Added new functions: toggleDiffPodColor(), toggleDiffVerify(id), diffPodColorClick(), closeDiffInlineColorPicker(), resetDiffVerified(), updateDiffLegend()
- Modified openDiffModal() to include second toolbar with "Color POD" toggle, "Reset verify" button, verified counter, and legend bar
- Modified drawCell() to dim verified containers with faded POD color, green border, and checkmark
- Added click handler on diff rows for container selection and scrolling
- Added document click handler to close inline color picker on outside clicks
- Updated buildExportData() with Mismo_Hatch and Verificado columns
- Updated resetAllFilters() to clear DIFF_VERIFIED and DIFF_POD_COLOR_MODE
- Updated help documentation with new diff panel features

Stage Summary:
- Diff log now shows POD color dots next to each container, clickable to change color
- "Color POD" button toggles row background coloring by POD
- Background differentiates same-hatch (orange tint) vs different-hatch (purple tint) moves
- Each row has a clickable verification tick that dims the row and the container in the canvas
- Verified containers show faded POD color with green border and checkmark
- Export includes new Mismo_Hatch and Verificado columns
- All features documented in help modal
