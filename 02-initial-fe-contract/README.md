# 02 — Anton's 1st FE JSON (Initial Contract)

- **Date:** February 4, 2026
- **Author:** Anton Konovalov (Team B / Backend)
- **Channel:** `#prelim-product` (initial post), DMs with Ahmed

---

## Context

Anton posted the first `FloorplanEditorSketch` JSON to `#prelim-product` on Feb 4. This was the **first concrete data contract proposal** between Team BE (Anton's converter) and Team Core (Ahmed's auto-estimate engine).

At this point Anton was clear: *"It is not finalized, I'm working on adding more things there. We're working on MVP."* and *"We don't have any final contracts yet. That JSON I sent should be used as contact between Team BE and Team Core."*

Ahmed responded with 4 clarification questions that kicked off the iterative contract negotiation:

1. How to compute total floor area (answer: sum `room.area`, units are cm²)
2. Will the schema include `waterCategory`, `waterClass`, treatment info/material? (answer: will add water fields, treatment not in app yet)
3. Plan to add `roomId` to equipment and wallMoisture? (answer: can add)
4. Use `room.roomName` or `placements[room.placement].name`? (answer: use `roomName`)

---

## Files

### `FE_full.json`

The first FloorplanEditorSketch JSON example. Structure:

- 7 walls, 14 corners, 1 room ("Example2")
- 3 doors, 2 windows
- 8 equipment items (**no `roomId` yet**)
- 3 shapes (2 moisture area polygons + 1 barrier) — **no `type` or `affectedSurfaces` fields yet**
- 1 wall moisture entry
- Settings: `wallHeight: 270cm`, `measurementUnit: "cm"`

**What was missing at this stage:**

- No `roomId` on equipment
- No `projectData` (no `waterCategory` / `waterClass`)
- No `type` or `affectedSurfaces` on shapes
- No `length` on walls
- No enum normalization

### `floorplan_editor.md`

The full specification document for the `FloorplanEditorSketch` JSON format. This is the **canonical reference** for the FE JSON structure, covering:

- Coordinate system (all in centimeters, origin from iOS/ARKit 3D space)
- Wall-corner relationship model (corners reference walls via `wallStarts`/`wallEnds`)
- Door types, window structure, holes (openings)
- Room definitions (corner-based polygons, area in cm²)
- Equipment types (`dehumidifier`, `Air Mover`, `Air Scrubber`)
- Shape geometry (dots + lines forming closed polygons, composite IDs)
- Containment barriers (shapes with `type: "barrier"`)
- Wall moisture (normalized position along wall, height above 2ft baseline, doorway exclusion ranges)
- Complete minimal JSON example

---

## What Changed After This

All 4 of Ahmed's requests were addressed in subsequent versions:

- `roomId` → added in folder 03 (Feb 6, after bug fix)
- `waterCategory` / `waterClass` → added Feb 5, mobile bug fixed Feb 6
- `shapes.type` + `affectedSurfaces` → added Feb 6
- `wall.length` → added Feb 9
