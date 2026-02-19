# 04 — Anton's Last FE: Combined JSON Prototype

- **Date:** February 16, 2026
- **Author:** Anton Konovalov (Team B / Backend)
- **Channel:** `#prelim-product`, DMs with Ahmed

---

## Context

On Feb 16, the conversation shifted from the FE-only JSON to a **combined structure**. Anton explained that the FE.json only contains Instant Sketch data converted to Floorplan Editor format, but the SQS input message also needs to include the **main DocuSketch project data** (panoramas, tour info, maps, attachments).

Anton's message: *"FE.json contains only instant sketch data converted to Floorplan Editor JSON. We also need to have a main DocuSketch project data there. The idea is to combine them both in one JSON. I'm trying to understand how to perform that structure in the best way."*

This folder contains Anton's first prototype of that combined structure.

---

## Files

### `Untitled.json`

Prototype of the combined JSON structure. This is a **fundamentally different format** from the previous FE.json files — it wraps the Docusketch tour/panorama model:

```JSON
{
  "mainTour": {
    "id": "<ObjectId>",
    "name": "Project Name",
    "maps": {
      "1": {
        "rooms": { ... },       // rooms with hotspots (SVG overlays, pano links)
        "polygons": { ... }     // room boundary polygons by roomId
      }
    },
    "panos": [                  // 360 panorama images
      {
        "id": "...",
        "name": "Living room",
        "originalImage": "https://...",
        "tiles": { ... },       // multi-resolution tile levels for rendering
        "cameraHeight": 5.25,
        "attachments": {        // damage documentation (photos, videos, notes)
          "attachment_Id": {
            "content": [
              { "type": "CAUSE_OF_LOSS", "url": "...", "videoUrl": "..." },
              { "type": "NOTE", "description": "...", "url": "..." }
            ]
          }
        }
      }
    ]
  },
  "timelineTours": []           // empty — for future timeline-based snapshots
}
```

**What's new here:**

- `mainTour` — wraps the entire Docusketch project with panoramas, maps, and attachments
- `panos[]` — 360 panorama images with multi-resolution tiles, camera height, and attachments containing damage evidence (photos, videos, notes with types like `CAUSE_OF_LOSS`, `NOTE`)
- `maps` — room polygons and hotspots linking panoramas to floorplan positions
- `timelineTours` — placeholder for timeline snapshots (empty for now)

**What's NOT in this prototype yet:**

- The Instant Sketch restoration data (equipment, moisture areas, barriers, wall moisture) — Anton's last message noted this is the open design question

This is the active design decision as of Feb 18, 2026.

---

## Open Design Decision

Where does the Instant Sketch restoration data go in the combined structure?

| Option | Approach | Trade-off |
|--------|----------|-----------|
| **A — Flat top-level entities** | `equipment[]`, `moistureAreas[]`, `barriers[]` as siblings to `mainTour`, linked by `roomId`/`wallId` | Clean for consumers, but two different paradigms in one payload |
| **B — Nested in rooms** | Inside `maps.<N>.rooms.<roomId>` | Co-located, but wall moisture and barriers don't belong to a single room |
| **C — Dedicated `floorplan` namespace** | `"floorplan": { walls, rooms, equipment, ... }` as a top-level sibling to `mainTour` | Clean separation, merge at envelope level only |

Decision pending — Ahmed and Anton are actively discussing this.
