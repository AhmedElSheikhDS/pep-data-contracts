# 03 — Anton's 2nd FE JSON (After BE Core Alignment)

- **Date:** February 6–12, 2026
- **Author:** Anton Konovalov (Team B / Backend)
- **Channel:** DMs with Ahmed, `#estimate_strategycal_team_public`

---

## Context

This folder captures the FE JSON **after Anton aligned with the backend/core team** and addressed the requests from Ahmed's initial review. Between Feb 5–12, several key changes were made:

- **Feb 5:** `projectData.waterCategory` and `projectData.waterClass` added. Anton flagged a bug with `roomId` calculation for equipment.
- **Feb 6 (09:43):** Anton sends corrected JSON with `roomId` on equipment, multi-room support.
- **Feb 6 (13:06):** `waterCategory`/`waterClass` missing from the file — traced to a mobile app bug. Anton escalated to the mobile developer.
- **Feb 6 (15:05):** Anton sends updated JSON with `waterCategory`/`waterClass` working, plus new `type: "moistureArea"` and `affectedSurfaces` fields on shapes.
- **Feb 9:** `length` property added to all walls.
- **Feb 10:** Ahmed confirms this JSON as the agreed input contract for the async endpoint.
- **Feb 12:** Enum normalization — `affectedSurfaces` changed from single string to array with capitalized values, equipment types capitalized with underscores.

---

## Files

### `FE.json`

The second-generation FloorplanEditorSketch JSON. Key differences from folder 02:

| Change | Before (02) | After (03) |
|--------|-------------|------------|
| Rooms | 1 room ("Example2") | 3 rooms ("First Drying Chamber 1/2/3") |
| Equipment `roomId` | Not present | Present on all equipment |
| `projectData` | Not present | `waterCategory: "CAT 3"`, `waterClass: "Class 4"` |
| Shapes `type` | Not present | `"moistureArea"` |
| Shapes `affectedSurfaces` | Not present | `"Floor"` |
| Wall `length` | Not present | Present on all walls |
| Corner `wallStarts`/`wallEnds` | ID-only references | Full embedded wall objects |
| Room `corners` | ID-only references | Full embedded corner objects |
| Placements keys | UUID | Room name strings |

Structure:

- 10 walls (with `length`), 9 corners
- 3 rooms (areas: 200,000 / 120,000 / 90,000 cm²)
- 2 doors, 3 windows
- 5 equipment items (3 Air Movers, 1 Dehumidifier, 1 Air Scrubber — all with `roomId`)
- 7 shapes (moisture areas with `type` and `affectedSurfaces`)
- No wall moisture entries in this particular example
- Settings: `wallHeight: 270cm`, `measurementUnit: "cm"`

---

## Decisions Made in This Phase

1. **`room.roomName` is the source of truth** for room names (not placements lookup).
2. **`room.area` is in square centimeters** — sum all rooms for total floor area.
3. **`roomId` on wallMoisture deferred** — the mobile app applies wall moisture to whole walls, not room-specific edges. May be fixed later.
4. **Treatment info/material** escalated to Oleg Shultsev (mobile team) — not available in the app yet.
5. **Wall length confirmed as needed** by the team and domain expert (Feb 6 daily standup).
6. **This JSON confirmed as the input contract** for the async endpoint (Feb 10).
