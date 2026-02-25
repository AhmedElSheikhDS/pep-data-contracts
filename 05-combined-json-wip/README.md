# 05 — Combined JSON (WIP)

- **Date:** February 24, 2026
- **Source:** Anton Konovalov (Team BE / Face)
- **File:** `24-feb-Untitled.json`

---

## Changes from Previous FE.json Versions

- **`maps` removed** — replaced by direct `roomId` on each pano (cleaner linkage)
- **`mainTour` added** — wraps DocuSketch tour with panos, attachments, typeOfLoss
- **`timelineTours` added** — empty placeholder for future use
- **`moistureAreas`** used directly instead of `shapes` with `type: "moistureArea"`
- **`placements`** added — room metadata lookup (roomType, isBalcony)
- **`settings`** added — measurementUnit, wallHeight, floorHeight
- **`lines`/`dots`** at top level — freeform geometry, duplicated inside moistureAreas/barriers

### Feb 12 Normalization Changes — Confirmed

Both schema alignment requests from Feb 12 have been applied:

1. **`affectedSurfaces`** — now an array with capitalised enum values (e.g., `["Floor"]`). Previously was a single string.
2. **Equipment types** — capitalised, no spaces, enum-friendly: `Air_Mover`, `Air_Scrubber`, `Dehumidifier`.

---

## Context

Anton sent the first real combined JSON that merges the DocuSketch tour data (`mainTour`) with the Instant Sketch restoration data (`instantSketch`) into a single flat payload. This follows the structure Ahmed proposed on Feb 16 and resolves the earlier design decision from `04-combined-json-prototype/`.

The file is WIP because several fields are still missing or empty (material/treatment, wallMoisture, saturation level, dehumidifier type). Anton confirmed the structure is correct but noted that a realistic example depends on mobile developers capturing data with 360 cameras and LiDAR.

---

## Structure

Three top-level keys:

| Key | Type | Description |
|-----|------|-------------|
| `mainTour` | Object | DocuSketch project: panoramas, tour ID, loss type |
| `timelineTours` | Array | Empty — placeholder for future timeline snapshots |
| `instantSketch` | Object | Floorplan Editor data: rooms, walls, equipment, moisture, barriers |

### `mainTour`

| Field | Description |
|-------|-------------|
| `id` | Project ID |
| `typeOfLoss` | `{"type": "WATER"}` — loss classification |
| `panos[]` | 6 panoramas, each with `id`, `name`, `roomId`, images, tiles, `cameraHeight`, and optional `attachments` |

Each pano's `roomId` links to `instantSketch.rooms[]` — this is how panoramas connect to sketch rooms.

Pano `attachments` contain technician field notes (type `NOTE`) with free-text damage descriptions. Example:
> "Bathroom: 100% tile, Detach toilet, 100% quarter round, 100% baseboards, 50% Wainscoting, 100% treat, 100% mortar board, Set three and one"

### `instantSketch`

| Field | Type | Count | Notes |
|-------|------|-------|-------|
| `rooms` | Array | 3 | `id`, `roomName`, `area` (cm^2), `corners[]`, `placement` |
| `walls` | Array | 10 | `id`, `thickness`, `length` (cm), `bearing`, `orphan` |
| `corners` | Array | 8 | `id`, `x`, `y`, `wallStarts[]`, `wallEnds[]` |
| `doors` | Array | 2 | Position coordinates, `type`, `direction`, `height` |
| `windows` | Array | 3 | Position, `wall` ref, `height`, `heightFromFloor`, `length` |
| `equipment` | Array | 5 | `type`, `description`, `coordinates`, `roomId` |
| `moistureAreas` | Array | 1 | Polygon of dots/lines, `height`, `affectedSurfaces` |
| `barriers` | Array | 1 | Containment barrier dots/lines, `height` |
| `wallMoisture` | Array | 0 | **Empty** |
| `placements` | Object | 3 | Room metadata: `name`, `roomType`, `isBalcony` |
| `projectData` | Object | — | `waterCategory`: "CAT 2", `waterClass`: "Class 3" |
| `settings` | Object | — | `measurementUnit`: "cm", `wallHeight`: 270, `floorHeight`: 0 |
| `lines` | Array | 24 | Freeform line segments (also embedded in moistureAreas/barriers) |
| `dots` | Array | 25 | Freeform points (also embedded in moistureAreas/barriers) |
| `holes` | Array | 0 | Empty |
| `arcs` | Array | 0 | Empty |
| `cameras` | Array | 0 | Empty |
| `embeds` | Array | 0 | Empty |
| `pois` | Array | 0 | Empty |
| `kitchens` | Array | 0 | Empty |

---

## Available Raw Data (for line item generation)

| Raw Data | Source Field | Unit |
|----------|-------------|------|
| Room names | `instantSketch.rooms[].roomName` | — |
| Room area | `instantSketch.rooms[].area` | cm^2 |
| Room dimensions | Derivable from `rooms[].corners[]` coordinates | cm |
| Wall lengths | `instantSketch.walls[].length` | cm |
| Wall height | `instantSketch.settings.wallHeight` | cm |
| Equipment type & count per room | `instantSketch.equipment[].type` + `roomId` | — |
| Moisture area geometry | `instantSketch.moistureAreas[].dots[]` | cm |
| Affected surfaces | `instantSketch.moistureAreas[].affectedSurfaces` | enum: Floor, Ceiling, Wall, Floor & Ceiling |
| Containment barriers | `instantSketch.barriers[]` | — |
| Water category | `instantSketch.projectData.waterCategory` | "CAT 1" / "CAT 2" / "CAT 3" |
| Water class | `instantSketch.projectData.waterClass` | "Class 1" .. "Class 4" |
| Loss type | `mainTour.typeOfLoss.type` | "WATER" |
| Technician field notes | `mainTour.panos[].attachments[].content[].description` | free text |
| Doors & windows | `instantSketch.doors[]`, `instantSketch.windows[]` | — |

---

## Missing / Empty (WIP gaps)

| Field | Status | Impact |
|-------|--------|--------|
| Material per surface | Not in structured fields; only in free-text pano notes | Blocks treatment line items |
| Wall moisture readings | `wallMoisture[]` is empty | Cannot generate moisture-based line items |
| Saturation level | Derivable from `wallMoisture` once populated | — |
| Dehumidifier type (LGR vs conventional) | Equipment `type` is generic "Dehumidifier" — asked Anton | Affects equipment line items |
| containToRoom | Derivable from barrier geometry + room boundaries | — |

Material/treatment is pending — Ahmed is pushing Oleg Shultsev to add it to the Instant Sketch mobile app.

---

## Linkage: `mainTour.panos` <-> `instantSketch.rooms`

Each pano has a `roomId` that links to `instantSketch.rooms[].id`. In this test file, all 6 panos point to the same room ("First Drying Chamber 1") — this is a test data artifact. Anton confirmed the linkage mechanism is correct.

---

## Entity Counts

| Entity | Count |
|--------|-------|
| Rooms | 3 ("First Drying Chamber 1", "First Drying Chamber 2", "First Drying Chamber 3") |
| Walls | 10 |
| Corners | 8 |
| Doors | 2 |
| Windows | 3 |
| Equipment | 5 (3x Air_Mover, 1x Air_Scrubber, 1x Dehumidifier) |
| Moisture Areas | 1 (Floor) |
| Barriers | 1 |
| Panos | 6 |
