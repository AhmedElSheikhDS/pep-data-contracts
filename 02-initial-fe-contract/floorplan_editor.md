# FloorplanEditorSketch JSON Format Specification

This document describes the JSON structure used by the Floorplan Editor application. This format is generated from Instant Sketch App data and used for rendering and editing 2D floor plans.

## Table of Contents

- [Overview](#overview)
- [Coordinate System](#coordinate-system)
- [Top-Level Structure](#top-level-structure)
- [Walls](#walls)
- [Corners](#corners)
- [Doors](#doors)
- [Windows](#windows)
- [Holes (Openings)](#holes-openings)
- [Rooms](#rooms)
- [Placements](#placements)
- [Equipment](#equipment)
- [Shapes](#shapes)
- [Containment Barriers](#containment-barriers)
- [Wall Moisture](#wall-moisture)
- [Dots and Lines](#dots-and-lines)
- [JSON Example](#json-example)

---

## Overview

`FloorplanEditorSketch` is the main data structure representing a complete floor plan. It contains all geometric elements (walls, corners), openings (doors, windows, holes), rooms, and damage-related markers (moisture areas, equipment, containment barriers).

The format is designed to be compatible with the Floorplan Editor web application's `FloorModel.getCurrentState()` output.

---

## Coordinate System

| Property | Unit | Description |
|----------|------|-------------|
| All coordinates (x, y) | **Centimeters (cm)** | All spatial coordinates are in centimeters |
| Heights | **Centimeters (cm)** | Wall height, door height, window height, shape height |
| Angles | **Radians** | Compass angle, camera angle |
| Normalized positions | **0.0 - 1.0** | Position along wall for moisture markers, barrier positions |

### Coordinate Origin
- Origin (0, 0) is the reference point converted from iOS/ARKit 3D space
- X increases to the right
- Y increases downward (2D projection of 3D Z-axis, negated)

---

## Top-Level Structure

```json
{
  "walls": [],           // Array of EditorWall
  "corners": [],         // Array of EditorCorner
  "arcs": [],            // Array of EditorArc (curved wall segments)
  "rooms": [],           // Array of EditorRoom
  "doors": [],           // Array of EditorDoor
  "windows": [],         // Array of EditorWindow
  "holes": [],           // Array of EditorHole (openings)
  "cameras": [],         // Array of EditorCamera (panorama positions)
  "embeds": [],          // Array of EditorEmbed (embedded 3D objects)
  "placements": {},      // Map<String, EditorPlacement> (room type metadata)
  "lines": [],           // Array of EditorLine (shape edges)
  "dots": [],            // Array of EditorDot (shape vertices)
  "shapes": [],          // Array of EditorShape (moisture areas, barriers)
  "pois": [],            // Array of EditorPoi (points of interest)
  "equipment": [],       // Array of EditorEquipment (restoration equipment)
  "wallMoisture": [],    // Array of EditorWallMoisture
  "compass": {},         // EditorCompass (north direction)
  "settings": {},        // EditorSettings (global settings)
  "kitchens": []         // Array of EditorKitchen
}
```

---

## Walls

Walls are the primary structural elements. Each wall connects two corners.

### EditorWall Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `thickness` | Double | Wall thickness in cm (default: 15 cm) |
| `bearing` | Integer | Wall mode/type (see Wall Modes below) |
| `height` | Double | Wall height in cm (optional) |
| `orphan` | Boolean | True if wall is not connected to a room |
| `arcGroupId` | String | ID of arc group if wall is curved (optional) |
| `connectsRoom1` | String | ID of first connected room (optional) |
| `connectsRoom2` | String | ID of second connected room (optional) |
| `connectsBothWays` | Boolean | Whether wall connects rooms in both directions |
| `isCreatedIn2D` | Boolean | Whether wall was created in 2D editor |
| `leadsToRoom` | String | Room ID this wall leads to (optional) |

### Wall Modes (bearing values)

| Value | Mode | Description |
|-------|------|-------------|
| 0 | NORMAL | Standard wall |
| 1 | THICK | Thick wall (exterior, structural) |
| 2 | THIN | Thin wall (partition) |
| 3 | INVISIBLE | Invisible wall (room divider) |
| 4 | CUSTOM | Custom thickness |
| 5 | MISSING | Missing/removed wall |

### Wall-Corner Relationship

Walls don't directly store corner coordinates. Instead:
- Corner position is stored in `EditorCorner`
- Corner references walls via `wallStarts` and `wallEnds` arrays
- A wall starts at one corner and ends at another

**Important**: The `start` and `end` fields on `EditorWall` are **not serialized** to JSON. The wall-corner relationship is established through the Corner's `wallStarts`/`wallEnds` arrays.

### Wall Example

```json
{
  "id": "wall-123",
  "thickness": 15.0,
  "bearing": 0,
  "orphan": false
}
```

---

## Corners

Corners are vertices where walls meet. They define wall endpoints and room boundaries.

### EditorCorner Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `x` | Double | X coordinate in cm |
| `y` | Double | Y coordinate in cm |
| `wallStarts` | Array<EditorWallRef> | Walls that **start** at this corner |
| `wallEnds` | Array<EditorWallRef> | Walls that **end** at this corner |
| `arcGroupId` | String | ID of arc group if corner is on a curve (optional) |

### EditorWallRef Structure

Simple reference containing only the wall ID:

```json
{
  "id": "wall-123"
}
```

### Corner Example

```json
{
  "id": "corner-456",
  "x": 100.0,
  "y": 200.0,
  "wallStarts": [
    { "id": "wall-123" },
    { "id": "wall-789" }
  ],
  "wallEnds": [
    { "id": "wall-456" }
  ]
}
```

### How Walls and Corners Connect

```
Corner A (wallStarts: [wall-1]) ----[wall-1]---- Corner B (wallEnds: [wall-1])
         x: 0, y: 0                                    x: 200, y: 0
```

To find wall endpoints:
1. Find all corners
2. For each corner, check `wallStarts` - this corner is the **start** of those walls
3. For each corner, check `wallEnds` - this corner is the **end** of those walls

---

## Doors

Doors are openings in walls that allow passage between rooms.

### EditorDoor Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `x1` | Double | Start X coordinate in cm |
| `y1` | Double | Start Y coordinate in cm |
| `x2` | Double | End X coordinate in cm |
| `y2` | Double | End Y coordinate in cm |
| `double` | Boolean | True if this is a double door |
| `wall` | Object/null | Reference to parent wall (optional) |
| `wallWidth` | Double | Thickness of containing wall in cm |
| `startDistance` | Double | Distance from wall start to door |
| `type` | Integer | Door type (see Door Types below) |
| `direction` | Integer | Swing direction (1 = forward, -1 = backward) |
| `pathFrom` | Integer | Side from which door opens |
| `openRate` | Double | How open the door is (0.0 - 1.0) |
| `height` | Double | Door height in cm (default: 210 cm) |
| `leadsToRoom` | String | Room ID this door leads to (optional) |
| `connectsRoom1` | String | First connected room ID (optional) |
| `connectsRoom2` | String | Second connected room ID (optional) |
| `connectsBothWays` | Boolean | Whether door connects rooms bidirectionally |
| `isCreatedIn2D` | Boolean | Whether created in 2D editor |

### Door Types

| Value | Type | Description |
|-------|------|-------------|
| 1 | TYPE_WITH_PATH | Standard hinged door with swing path |
| (others) | - | Additional types for sliding, pocket, etc. |

### Door Example

```json
{
  "id": "door-001",
  "x1": 50.0,
  "y1": 100.0,
  "x2": 140.0,
  "y2": 100.0,
  "double": false,
  "wallWidth": 15.0,
  "type": 1,
  "direction": 1,
  "pathFrom": 1,
  "openRate": 1.0,
  "height": 210.0
}
```

The door width is calculated as: `sqrt((x2-x1)² + (y2-y1)²)`

---

## Windows

Windows are openings in walls at a certain height above the floor.

### EditorWindow Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `x1` | Double | Start X coordinate in cm |
| `y1` | Double | Start Y coordinate in cm |
| `x2` | Double | End X coordinate in cm |
| `y2` | Double | End Y coordinate in cm |
| `wall` | Object/null | Reference to parent wall (optional) |
| `wallWidth` | Double | Thickness of containing wall in cm |
| `startDistance` | Double | Distance from wall start to window |
| `height` | Double | Window height in cm (default: 100 cm) |
| `heightFromFloor` | Double | Distance from floor to window bottom in cm (default: 90 cm) |
| `length` | Double | Window length (optional) |
| `isCreatedIn2D` | Boolean | Whether created in 2D editor |

### Window Example

```json
{
  "id": "window-001",
  "x1": 200.0,
  "y1": 0.0,
  "x2": 320.0,
  "y2": 0.0,
  "wallWidth": 15.0,
  "height": 100.0,
  "heightFromFloor": 90.0
}
```

---

## Holes (Openings)

Holes represent open passages through walls (no door, just an opening).

### EditorHole Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `x1` | Double | Start X coordinate in cm |
| `y1` | Double | Start Y coordinate in cm |
| `x2` | Double | End X coordinate in cm |
| `y2` | Double | End Y coordinate in cm |
| `wall` | Object/null | Reference to parent wall (optional) |
| `wallWidth` | Double | Thickness of containing wall in cm |
| `startDistance` | Double | Distance from wall start to hole |
| `height` | Double | Hole height in cm (default: 210 cm) |
| `heightFromFloor` | Double | Distance from floor to hole bottom (default: 0) |
| `length` | Double | Hole length (optional) |
| `isCreatedIn2D` | Boolean | Whether created in 2D editor |

### Hole Example

```json
{
  "id": "hole-001",
  "x1": 400.0,
  "y1": 150.0,
  "x2": 500.0,
  "y2": 150.0,
  "wallWidth": 15.0,
  "height": 210.0,
  "heightFromFloor": 0.0
}
```

---

## Rooms

Rooms are enclosed spaces defined by their corner vertices.

### EditorRoom Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `roomName` | String | Display name of the room |
| `corners` | Array<String> | Ordered list of corner IDs defining room boundary |
| `area` | Double | Room area in cm² |
| `placement` | String | ID of associated EditorPlacement |

### Room Example

```json
{
  "id": "room-001",
  "roomName": "Living Room",
  "corners": ["corner-1", "corner-2", "corner-3", "corner-4"],
  "area": 2000000.0,
  "placement": "placement-001"
}
```

**Note**: Area is in square centimeters. To convert to square meters: `area / 10000`

---

## Placements

Placements provide metadata about room types and characteristics.

### EditorPlacement Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `name` | String | Display name |
| `roomType` | String | Room type classification |
| `isBalcony` | Boolean | True if this is a balcony/exterior space |

### Placement Example

```json
{
  "placement-001": {
    "id": "placement-001",
    "name": "Living Room",
    "roomType": "room",
    "isBalcony": false
  }
}
```

**Note**: Placements is a **Map** (object), not an array. Keys are placement IDs.

---

## Equipment

Equipment represents restoration equipment placed on the floor plan for water damage restoration projects.

### EditorEquipment Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `type` | String | Equipment type identifier |
| `description` | String | Human-readable equipment description |
| `coordinates` | Object | Position on floor plan |
| `coordinates.x` | Double | X coordinate in cm |
| `coordinates.y` | Double | Y coordinate in cm |

### Equipment Types

| Type | Description |
|------|-------------|
| `dehumidifier` | Dehumidifier |
| `Air Mover` | Air Mover (fan) |
| `Air Scrubber` | Air Scrubber (air filtration) |

### Equipment Example

```json
{
  "id": "equip-001",
  "type": "dehumidifier",
  "description": "Dehumidifier",
  "coordinates": {
    "x": 150.0,
    "y": 250.0
  }
}
```

---

## Shapes

Shapes represent closed polygons on the floor plan, used for moisture areas, demolition zones, and containment barriers.

### EditorShape Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Composite UUID (generated from sorted dot IDs) |
| `dots` | Array<EditorDot> | Vertices of the polygon |
| `lines` | Array<EditorLine> | Edges connecting the vertices |
| `height` | Double | Z-axis height in cm (default: 90 cm) |
| `type` | String/null | Shape type (null = regular shape, "barrier" = containment barrier) |

### Shape ID Generation

The shape ID is a composite string generated by:
1. Collecting all dot IDs
2. Sorting them alphabetically
3. Joining with commas

Example: `"dot-123,dot-456,dot-789"`

### Shape Example (Regular Moisture Area)

```json
{
  "id": "dot-a,dot-b,dot-c,dot-d",
  "dots": [
    { "id": "dot-a", "x": 100.0, "y": 100.0 },
    { "id": "dot-b", "x": 200.0, "y": 100.0 },
    { "id": "dot-c", "x": 200.0, "y": 200.0 },
    { "id": "dot-d", "x": 100.0, "y": 200.0 }
  ],
  "lines": [
    { "id": "line-1", "start": {"id": "dot-a", "x": 100.0, "y": 100.0}, "end": {"id": "dot-b", "x": 200.0, "y": 100.0}, "thickness": 5.0 },
    { "id": "line-2", "start": {"id": "dot-b", "x": 200.0, "y": 100.0}, "end": {"id": "dot-c", "x": 200.0, "y": 200.0}, "thickness": 5.0 },
    { "id": "line-3", "start": {"id": "dot-c", "x": 200.0, "y": 200.0}, "end": {"id": "dot-d", "x": 100.0, "y": 200.0}, "thickness": 5.0 },
    { "id": "line-4", "start": {"id": "dot-d", "x": 100.0, "y": 200.0}, "end": {"id": "dot-a", "x": 100.0, "y": 100.0}, "thickness": 5.0 }
  ],
  "height": 90.0,
  "type": null
}
```

---

## Containment Barriers

Containment barriers are special shapes with `type: "barrier"`. They represent physical barriers that block water migration pathways during restoration.

### Barrier Representation

Unlike regular shapes (closed polygons), barriers are **lines** represented as a shape with only 2 dots connected by 1 line.

### Barrier Types (from source data)

| Barrier Type | Description |
|--------------|-------------|
| `doorwaySpan` | Barrier spanning across a doorway opening |
| `wallToWall` | Barrier running from one wall to another wall |

### Containment Barrier Example

```json
{
  "id": "barrier-001",
  "dots": [
    { "id": "dot-start", "x": 50.0, "y": 100.0 },
    { "id": "dot-end", "x": 140.0, "y": 100.0 }
  ],
  "lines": [
    {
      "id": "line-barrier",
      "start": { "id": "dot-start", "x": 50.0, "y": 100.0 },
      "end": { "id": "dot-end", "x": 140.0, "y": 100.0 },
      "thickness": 5.0
    }
  ],
  "height": 90.0,
  "type": "barrier"
}
```

### Identifying Barriers

To identify containment barriers in the shapes array:
```javascript
const barriers = shapes.filter(shape => shape.type === "barrier");
const moistureAreas = shapes.filter(shape => shape.type !== "barrier");
```

---

## Wall Moisture

Wall moisture markers indicate areas of moisture detection on walls, used for water damage assessment.

### EditorWallMoisture Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `wallId` | String | ID of the wall this moisture marker is on |
| `moistureHeightAbove2Ft` | Double | Height of moisture above 2-foot baseline (in feet) |
| `isWholeWall` | Boolean | True if moisture covers entire wall height |
| `positionAlongWall` | Double | Start position along wall (0.0 - 1.0) |
| `lengthAlongWall` | Double | Coverage length along wall (0.0 - 1.0) |
| `doorwayRanges` | Array<DoorwayRange> | Doorway sections to exclude |
| `startCoordinates` | Coordinates | Computed start point in cm |
| `endCoordinates` | Coordinates | Computed end point in cm |

### DoorwayRange Structure

| Field | Type | Description |
|-------|------|-------------|
| `start` | Double | Start of doorway along wall (0.0 - 1.0) |
| `end` | Double | End of doorway along wall (0.0 - 1.0) |

### Coordinates Structure

| Field | Type | Description |
|-------|------|-------------|
| `x` | Double | X coordinate in cm |
| `y` | Double | Y coordinate in cm |

### Position Along Wall

The `positionAlongWall` and `lengthAlongWall` fields use **normalized values**:
- `0.0` = start of wall
- `1.0` = end of wall
- `0.5` = middle of wall

Example: `positionAlongWall: 0.2, lengthAlongWall: 0.4` means moisture starts at 20% of wall length and covers 40% of the wall.

### Moisture Height Calculation

The `moistureHeightAbove2Ft` field indicates moisture height above a 2-foot baseline:
- Value of `0.0` = moisture at 2 feet high
- Value of `2.0` = moisture at 4 feet high (2 + 2)
- Value of `6.0` = moisture at 8 feet high (2 + 6)

If `isWholeWall` is `true`, the `moistureHeightAbove2Ft` value is ignored.

### Wall Moisture Example

```json
{
  "id": "moisture-001",
  "wallId": "wall-123",
  "moistureHeightAbove2Ft": 2.0,
  "isWholeWall": false,
  "positionAlongWall": 0.1,
  "lengthAlongWall": 0.6,
  "doorwayRanges": [
    { "start": 0.3, "end": 0.45 }
  ],
  "startCoordinates": { "x": 20.0, "y": 100.0 },
  "endCoordinates": { "x": 140.0, "y": 100.0 }
}
```

This represents moisture on wall-123:
- Starting at 10% of wall length
- Covering 60% of wall length (ending at 70%)
- 4 feet high (2 + 2)
- With a doorway exclusion between 30% and 45% of wall length

---

## Dots and Lines

Dots and Lines are the building blocks for shapes. They are stored both within each shape and at the top-level arrays for editor compatibility.

### EditorDot Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `x` | Double | X coordinate in cm |
| `y` | Double | Y coordinate in cm |

### EditorLine Structure

| Field | Type | Description |
|-------|------|-------------|
| `id` | String | Unique identifier (UUID) |
| `start` | EditorDot | Start vertex (embedded, not ID reference) |
| `end` | EditorDot | End vertex (embedded, not ID reference) |
| `thickness` | Double | Line thickness in pixels (default: 5.0) |

### Important Notes

1. **Dots/Lines are duplicated**: Each shape's dots and lines are **also** added to the top-level `dots` and `lines` arrays
2. **Lines embed full dots**: The `start` and `end` fields contain complete EditorDot objects, not just IDs
3. **Shapes are closed**: For closed shapes (moisture areas), lines form a complete loop

### Dot Example

```json
{
  "id": "dot-abc123",
  "x": 150.0,
  "y": 200.0
}
```

### Line Example

```json
{
  "id": "line-xyz789",
  "start": { "id": "dot-1", "x": 100.0, "y": 100.0 },
  "end": { "id": "dot-2", "x": 200.0, "y": 100.0 },
  "thickness": 5.0
}
```

---

## JSON Example

Complete minimal example with walls, a door, a room, equipment, moisture shape, wall moisture, and a containment barrier:

```json
{
  "walls": [
    { "id": "wall-1", "thickness": 15.0, "bearing": 0, "orphan": false },
    { "id": "wall-2", "thickness": 15.0, "bearing": 0, "orphan": false },
    { "id": "wall-3", "thickness": 15.0, "bearing": 0, "orphan": false },
    { "id": "wall-4", "thickness": 15.0, "bearing": 0, "orphan": false }
  ],
  "corners": [
    { "id": "c1", "x": 0.0, "y": 0.0, "wallStarts": [{"id": "wall-1"}], "wallEnds": [{"id": "wall-4"}] },
    { "id": "c2", "x": 300.0, "y": 0.0, "wallStarts": [{"id": "wall-2"}], "wallEnds": [{"id": "wall-1"}] },
    { "id": "c3", "x": 300.0, "y": 400.0, "wallStarts": [{"id": "wall-3"}], "wallEnds": [{"id": "wall-2"}] },
    { "id": "c4", "x": 0.0, "y": 400.0, "wallStarts": [{"id": "wall-4"}], "wallEnds": [{"id": "wall-3"}] }
  ],
  "arcs": [],
  "rooms": [
    { "id": "room-1", "roomName": "Living Room", "corners": ["c1", "c2", "c3", "c4"], "area": 1200000.0, "placement": "p1" }
  ],
  "doors": [
    { "id": "door-1", "x1": 100.0, "y1": 0.0, "x2": 190.0, "y2": 0.0, "type": 1, "direction": 1, "pathFrom": 1, "height": 210.0, "wallWidth": 15.0, "openRate": 1.0 }
  ],
  "windows": [
    { "id": "win-1", "x1": 50.0, "y1": 400.0, "x2": 150.0, "y2": 400.0, "height": 100.0, "heightFromFloor": 90.0, "wallWidth": 15.0 }
  ],
  "holes": [],
  "cameras": [],
  "embeds": [],
  "placements": {
    "p1": { "id": "p1", "name": "Living Room", "roomType": "room", "isBalcony": false }
  },
  "dots": [
    { "id": "d1", "x": 50.0, "y": 50.0 },
    { "id": "d2", "x": 100.0, "y": 50.0 },
    { "id": "d3", "x": 100.0, "y": 100.0 },
    { "id": "d4", "x": 50.0, "y": 100.0 },
    { "id": "b1", "x": 100.0, "y": 0.0 },
    { "id": "b2", "x": 190.0, "y": 0.0 }
  ],
  "lines": [
    { "id": "l1", "start": {"id": "d1", "x": 50.0, "y": 50.0}, "end": {"id": "d2", "x": 100.0, "y": 50.0}, "thickness": 5.0 },
    { "id": "l2", "start": {"id": "d2", "x": 100.0, "y": 50.0}, "end": {"id": "d3", "x": 100.0, "y": 100.0}, "thickness": 5.0 },
    { "id": "l3", "start": {"id": "d3", "x": 100.0, "y": 100.0}, "end": {"id": "d4", "x": 50.0, "y": 100.0}, "thickness": 5.0 },
    { "id": "l4", "start": {"id": "d4", "x": 50.0, "y": 100.0}, "end": {"id": "d1", "x": 50.0, "y": 50.0}, "thickness": 5.0 },
    { "id": "bl1", "start": {"id": "b1", "x": 100.0, "y": 0.0}, "end": {"id": "b2", "x": 190.0, "y": 0.0}, "thickness": 5.0 }
  ],
  "shapes": [
    {
      "id": "d1,d2,d3,d4",
      "dots": [
        { "id": "d1", "x": 50.0, "y": 50.0 },
        { "id": "d2", "x": 100.0, "y": 50.0 },
        { "id": "d3", "x": 100.0, "y": 100.0 },
        { "id": "d4", "x": 50.0, "y": 100.0 }
      ],
      "lines": [
        { "id": "l1", "start": {"id": "d1", "x": 50.0, "y": 50.0}, "end": {"id": "d2", "x": 100.0, "y": 50.0}, "thickness": 5.0 },
        { "id": "l2", "start": {"id": "d2", "x": 100.0, "y": 50.0}, "end": {"id": "d3", "x": 100.0, "y": 100.0}, "thickness": 5.0 },
        { "id": "l3", "start": {"id": "d3", "x": 100.0, "y": 100.0}, "end": {"id": "d4", "x": 50.0, "y": 100.0}, "thickness": 5.0 },
        { "id": "l4", "start": {"id": "d4", "x": 50.0, "y": 100.0}, "end": {"id": "d1", "x": 50.0, "y": 50.0}, "thickness": 5.0 }
      ],
      "height": 90.0,
      "type": null
    },
    {
      "id": "barrier-doorway-1",
      "dots": [
        { "id": "b1", "x": 100.0, "y": 0.0 },
        { "id": "b2", "x": 190.0, "y": 0.0 }
      ],
      "lines": [
        { "id": "bl1", "start": {"id": "b1", "x": 100.0, "y": 0.0}, "end": {"id": "b2", "x": 190.0, "y": 0.0}, "thickness": 5.0 }
      ],
      "height": 90.0,
      "type": "barrier"
    }
  ],
  "pois": [],
  "equipment": [
    { "id": "eq-1", "type": "dehumidifier", "description": "Dehumidifier", "coordinates": { "x": 200.0, "y": 200.0 } },
    { "id": "eq-2", "type": "Air Mover", "description": "Air Mover", "coordinates": { "x": 150.0, "y": 300.0 } }
  ],
  "wallMoisture": [
    {
      "id": "wm-1",
      "wallId": "wall-1",
      "moistureHeightAbove2Ft": 1.5,
      "isWholeWall": false,
      "positionAlongWall": 0.0,
      "lengthAlongWall": 0.3,
      "doorwayRanges": [],
      "startCoordinates": { "x": 0.0, "y": 0.0 },
      "endCoordinates": { "x": 90.0, "y": 0.0 }
    }
  ],
  "compass": { "angle": 0.0, "x": 250.0, "y": 50.0 },
  "settings": { "floorHeight": 0.0, "wallHeight": 270.0, "measurementUnit": "cm" },
  "kitchens": []
}
```

---

## Summary Table

| Element | Coordinate Fields | Units | Key Identifiers |
|---------|-------------------|-------|-----------------|
| Walls | (via corners) | cm | id, bearing |
| Corners | x, y | cm | id, wallStarts, wallEnds |
| Doors | x1, y1, x2, y2 | cm | id, type, direction |
| Windows | x1, y1, x2, y2 | cm | id, height, heightFromFloor |
| Holes | x1, y1, x2, y2 | cm | id, height |
| Rooms | (via corners) | cm² (area) | id, corners, placement |
| Equipment | coordinates.x, coordinates.y | cm | id, type, description |
| Shapes | dots[].x, dots[].y | cm | id (composite), type |
| Wall Moisture | startCoordinates, endCoordinates | cm | id, wallId, positionAlongWall |
| Barriers | dots[].x, dots[].y | cm | id, type="barrier" |
