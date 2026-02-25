# Input Contract Spec — Team BE (Face) -> Team Core

> **Version:** 0.1 (Draft)
> **Date:** February 25, 2026
> **Based on:** `05-combined-json-wip/24-feb-Untitled.json`
> **Direction:** Team BE (Anton) sends -> Team Core (Ahmed/Malte) consumes
> **Transport:** Amazon SQS (N. Virginia) — S3 fallback for payloads >256KB

---

## Envelope

The payload is a single JSON object with three top-level keys.

| Key | Type | Required | Description |
|-----|------|----------|-------------|
| `mainTour` | Object | Yes | DocuSketch project: panoramas, loss type, technician notes |
| `timelineTours` | Array | No | Reserved for future timeline snapshots. May be empty. |
| `instantSketch` | Object | Yes | Floorplan Editor data: rooms, walls, equipment, moisture, barriers |

---

## `mainTour`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String | Yes | DocuSketch project ID |
| `typeOfLoss` | Object | Yes | Loss classification |
| `typeOfLoss.type` | String | Yes | Enum: `"WATER"` (others TBD) |
| `panos` | Array\<Pano\> | Yes | 360 panorama images |

### Pano

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String | Yes | Pano ID |
| `name` | String | Yes | Pano name (e.g., "Hallway Bathroom 1st floor") |
| `roomId` | String | Yes | Links to `instantSketch.rooms[].id` |
| `originalImage` | String (URL) | Yes | Full-res panorama image |
| `originalImageRaw` | String (URL) | Yes | Raw unprocessed image |
| `thumbnail` | String (URL) | Yes | Thumbnail image |
| `plan` | Integer | Yes | Floor/plan number |
| `tiles` | Object | Yes | Multi-resolution tile data for rendering |
| `tiles.tileSize` | Integer | Yes | Tile dimension in px |
| `tiles.levels` | Array\<TileLevel\> | Yes | Resolution levels |
| `cameraHeight` | Float | Yes | Camera height (unit: feet) |
| `attachments` | Object | No | Keyed by spot ID. Contains technician notes and evidence. |

### Pano Attachment Content

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String | Yes | Attachment content ID |
| `type` | String | Yes | Enum: `"NOTE"`, `"CAUSE_OF_LOSS"`, others TBD |
| `description` | String | No | Free-text technician notes |
| `url` | String (URL) | No | Photo/document URL |
| `videoUrl` | String (URL) | No | Video URL |
| `name` | String | No | Attachment name |
| `comments` | String | No | Additional comments |
| `panorama` | String | No | Linked panorama reference |
| `previewUrl` | String (URL) | No | Preview image URL |
| `externalID` | String | No | External system reference |

---

## `instantSketch`

### `rooms[]`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String (UUID) | Yes | Room ID — referenced by `equipment[].roomId` and `mainTour.panos[].roomId` |
| `roomName` | String | Yes | Human-readable room name |
| `area` | Float | Yes | Room floor area. **Unit: cm^2** |
| `corners` | Array\<Corner\> | Yes | Room boundary corner coordinates |
| `placement` | String (UUID) | Yes | Links to `placements` object for room metadata |

### `walls[]`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String (UUID) | Yes | Wall ID |
| `thickness` | Float | Yes | Wall thickness. **Unit: cm** |
| `length` | Float | Yes | Wall length. **Unit: cm** |
| `bearing` | Integer | Yes | Wall orientation (degrees) |
| `orphan` | Boolean | Yes | Whether wall is disconnected from room boundaries |

### `corners[]`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String (UUID) | Yes | Corner ID |
| `x` | Float | Yes | X coordinate. **Unit: cm** |
| `y` | Float | Yes | Y coordinate. **Unit: cm** |
| `wallStarts` | Array\<WallRef\> | Yes | Walls starting at this corner |
| `wallEnds` | Array\<WallRef\> | Yes | Walls ending at this corner |

### `doors[]`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String (UUID) | Yes | Door ID |
| `x1`, `y1` | Float | Yes | Start position. **Unit: cm** |
| `x2`, `y2` | Float | Yes | End position. **Unit: cm** |
| `wallWidth` | Float | Yes | Width of wall containing this door. **Unit: cm** |
| `startDistance` | Float | Yes | Distance from wall start to door. **Unit: cm** |
| `type` | Integer | Yes | Door type code |
| `direction` | Integer | Yes | Opening direction |
| `pathFrom` | Integer | Yes | Path direction |
| `openRate` | Float | Yes | How far door opens (0.0–1.0) |
| `height` | Float | Yes | Door height. **Unit: cm** |

### `windows[]`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String (UUID) | Yes | Window ID |
| `x1`, `y1` | Float | Yes | Start position. **Unit: cm** |
| `x2`, `y2` | Float | Yes | End position. **Unit: cm** |
| `wall` | Object (WallRef) | Yes | Reference to containing wall |
| `wallWidth` | Float | Yes | Width of containing wall. **Unit: cm** |
| `startDistance` | Float | Yes | Distance from wall start. **Unit: cm** |
| `height` | Float | Yes | Window height. **Unit: cm** |
| `heightFromFloor` | Float | Yes | Sill height from floor. **Unit: cm** |
| `length` | Float | Yes | Window width. **Unit: cm** |
| `isCreatedIn2D` | Boolean | Yes | Whether created in 2D editor |

### `equipment[]`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String (UUID) | Yes | Equipment ID |
| `type` | String | Yes | Enum: `"Air_Mover"`, `"Air_Scrubber"`, `"Dehumidifier"` |
| `description` | String | Yes | Human-readable description |
| `coordinates` | Object | Yes | `{x, y}` position. **Unit: cm** |
| `coordinates.x` | Float | Yes | X position |
| `coordinates.y` | Float | Yes | Y position |
| `roomId` | String (UUID) | Yes | Links to `rooms[].id` |

**WIP:** Dehumidifier subtype (LGR vs conventional) not yet distinguished — asked Anton.

### `moistureAreas[]`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String | Yes | Composite ID (comma-separated dot IDs) |
| `dots` | Array\<Dot\> | Yes | Polygon vertices. Each: `{id, x, y}`. **Unit: cm** |
| `lines` | Array\<Line\> | Yes | Polygon edges. Each: `{id, start, end, thickness}` |
| `height` | Float | Yes | Height of affected area. **Unit: cm** |
| `type` | String | Yes | Always `"moistureArea"` |
| `affectedSurfaces` | Array\<String\> | Yes | Enum values: `"Floor"`, `"Ceiling"`, `"Wall"`, `"Floor & Ceiling"` |

### `barriers[]`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String (UUID) | Yes | Barrier ID |
| `dots` | Array\<Dot\> | Yes | Barrier endpoints. Each: `{id, x, y}`. **Unit: cm** |
| `lines` | Array\<Line\> | Yes | Barrier line segments |
| `height` | Float | Yes | Barrier height. **Unit: cm** |
| `type` | String | Yes | Always `"barrier"` |

### `wallMoisture[]`

Currently **empty** in the WIP file. Expected to contain per-wall moisture readings once mobile app provides data.

### `projectData`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `waterCategory` | String | Yes | Enum: `"CAT 1"`, `"CAT 2"`, `"CAT 3"` |
| `waterClass` | String | Yes | Enum: `"Class 1"`, `"Class 2"`, `"Class 3"`, `"Class 4"` |

### `settings`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `measurementUnit` | String | Yes | Unit system. Currently `"cm"` |
| `wallHeight` | Float | Yes | Default wall height. **Unit: cm** |
| `floorHeight` | Float | Yes | Floor elevation. **Unit: cm** |

### `placements`

Object keyed by room name. Each entry:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | String (UUID) | Yes | Placement ID — referenced by `rooms[].placement` |
| `name` | String | Yes | Room name (matches key) |
| `roomType` | String | Yes | Room type (e.g., `"room"`) |
| `isBalcony` | Boolean | Yes | Whether this is a balcony |

---

## Derived Fields (computed by Team Core, not in payload)

| Derived Field | Source | Logic |
|---------------|--------|-------|
| Saturation level | `wallMoisture[]` readings | High reading = saturated, low = dry |
| containToRoom | `barriers[].dots` + `rooms[].corners` | Geometric overlap determines which room is contained |
| Room perimeter | `rooms[].corners[]` coordinates | Sum of edge distances between sequential corners |
| Room dimensions (L x W) | `rooms[].corners[]` coordinates | Bounding box or edge analysis |
| Equipment count per room | `equipment[].roomId` | Group by roomId, count by type |
| Moisture area (sq cm) | `moistureAreas[].dots[]` | Polygon area calculation (shoelace formula) |

---

## WIP — Fields Not Yet in Schema

| Field | Owner | Status | Needed For |
|-------|-------|--------|------------|
| Material type per surface | Oleg Shultsev (Mobile) | Ahmed asked | Treatment line items |
| Wall moisture readings | Anton (Team BE) | Field exists, empty | Moisture-based line items, saturation derivation |
| Dehumidifier subtype | Anton (Team BE) | Ahmed asked | Equipment line items (LGR vs conventional pricing) |

---

## Validation Rules (Draft)

| Rule | Action |
|------|--------|
| `mainTour` missing | Reject |
| `instantSketch` missing | Reject |
| `rooms[]` empty | Reject |
| `projectData.waterCategory` missing | Reject |
| `projectData.waterClass` missing | Reject |
| `equipment[].roomId` not found in `rooms[].id` | Warn — orphaned equipment |
| `panos[].roomId` not found in `rooms[].id` | Warn — orphaned pano |
| `walls[].length` <= 0 | Warn — invalid wall |
| `rooms[].area` <= 0 | Warn — invalid room |
| Unknown fields | Accept — forward compatibility |
