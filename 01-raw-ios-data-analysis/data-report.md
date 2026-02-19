# Instant Sketch — TestFlight Data Export Analysis

**Report Date:** 2026-01-30
**Data Period:** 2026-01-14 to 2026-01-28 (14 days)
**Source:** Mixpanel TestFlight Analytics Export (iOS Instant Sketch App)
**Records:** 212 chamber events from 21 unique testers

---

## 1. Overview

| Metric | Value |
| --- | --- |
| Total Events | 212 |
| Unique Testers | 21 |
| Avg Events/Tester | ~10 |
| Countries | 7 |
| States/Provinces | 25 |
| Cities | 29 |
| Time Span | 14 days |
| Environment | `testflight` (100%) |

---

## 2. User Engagement

21 beta testers tracked via device-generated UUIDs (v7 format).

### Engagement Tiers

| Tier | Users | Events/User | Share of Events |
| --- | --- | --- | --- |
| Power Users | 3 | 35–48 | ~58% (122/212) |
| Regular | 5 | 10–22 | ~28% |
| Low Engagement | 13 | <10 | ~14% |

**Key Insight:** Top 3 users account for ~58% of all events. Most testers (13/21) submitted fewer than 10 chambers, suggesting casual or one-time trial usage.

---

## 3. Temporal Analysis

| Metric | Value |
| --- | --- |
| First Event | 2026-01-14 18:27 UTC |
| Last Event | 2026-01-28 16:39 UTC |
| Median Interval | ~4 minutes |
| Max Gap | ~27 hours |

Events occur in short bursts of activity (median 4 min apart), consistent with users scanning multiple rooms in a single session, with gaps of up to 27 hours between sessions.

---

## 4. Geographic Distribution

| Dimension | Unique Count |
| --- | --- |
| Countries | 7 |
| States/Provinces | 25 |
| Cities | 29 |

Users are distributed globally. Multiple cities per user suggest mobile/field usage, consistent with water damage restoration technicians visiting job sites.

---

## 5. Data Schema

| Column | Type | Description |
| --- | --- | --- |
| `timestamp` | DateTime | Event timestamp (UTC) |
| `distinct_id` | UUID v7 | Device/Tester ID |
| `environment` | String | Always `testflight` |
| `chamber_json` | JSON | Nested room scan + annotation data |
| `$geoip_city_name` | String | City from IP geolocation |
| `$geoip_subdivision_1_code` | String | State/Province code |
| `$geoip_country_code` | String | Country code |

---

## 6. Chamber JSON Structure

Each `chamber_json` contains a complete drying chamber: a LiDAR-scanned room plus user-annotated restoration data.

### 6.1 Top-Level Keys

| Key | Type | Present In | Description |
| --- | --- | --- | --- |
| `id` | UUID | 212/212 | Chamber ID |
| `chamberName` | String | 212/212 | User-given room name |
| `dehumidifierType` | String | 212/212 | Always `LGR` |
| `containToRoom` | Bool | 212/212 | Always `true` |
| `capturedRoom` | Object | 212/212 | LiDAR room scan data |
| `canvasOrigin` | Object | 212/212 | Canvas x/y (always `{5, 5}`) |
| `layerConfiguration` | Object | 203/212 | Active layer + state |
| `equipment` | Array | 78/212 (37%) | Placed equipment items |
| `floodCuts` | Array | 65/212 (31%) | Flood cut markings |
| `shapes` | Array | 55/212 (26%) | Moisture area shapes |
| `containmentBarriers` | Array | 43/212 (20%) | Barrier markings |
| `wallMoisture` | Array | 29/212 (14%) | Wall moisture readings |
| `roomNames` | Array | 14/212 (7%) | Named room sub-areas |
| `freeHandDrawings` | Array | 10/212 (5%) | Freehand moisture drawings |
| `waterCategory` | String | 120/212 (57%) | CAT 1/2/3 classification |
| `waterClass` | String | 120/212 (57%) | Class 1–4 classification |

### 6.2 Schema Variations

4 distinct key-set variations across records:

| Variant | Count | Distinguishing Keys |
| --- | --- | --- |
| Base + waterCategory/waterClass + layerConfig | 116 | Has all keys |
| Base + layerConfig (no water classification) | 87 | Missing `waterCategory`, `waterClass` |
| Base only (no layerConfig, no water class) | 5 | Missing `layerConfiguration` |
| Base + waterCategory/waterClass (no layerConfig) | 4 | Missing `layerConfiguration` |

---

## 7. Feature Adoption (Annotation Layers)

How often each annotation feature was used across all 212 chambers:

| Feature | Chambers Used | Adoption Rate | Total Items | Avg Items/Chamber (when used) |
| --- | --- | --- | --- | --- |
| Equipment | 78 | 37% | 485 | 6.2 |
| Flood Cuts | 65 | 31% | 209 | 3.2 |
| Shapes (moisture areas) | 55 | 26% | 88 | 1.6 |
| Containment Barriers | 43 | 20% | 100 | 2.3 |
| Wall Moisture | 29 | 14% | 143 | 4.9 |
| Room Names | 14 | 7% | 76 | 5.4 |
| Freehand Drawings | 10 | 5% | 16 | 1.6 |

**Key Insight:** Equipment placement is the most-used feature (37%), while freehand drawing is used rarely (5%). Wall moisture, despite low adoption (14%), has the highest density per use (4.9 items), suggesting it is used intensively when needed.

---

## 8. Annotation Value Breakdowns

### 8.1 Equipment Types (485 total items)

| Type | Count | Share |
| --- | --- | --- |
| Air Mover | 324 | 67% |
| Dehumidifier | 81 | 17% |
| Air Scrubber | 80 | 16% |

### 8.2 Containment Barrier Types (100 total items)

| Type | Count | Share |
| --- | --- | --- |
| Wall-to-Wall | 59 | 59% |
| Doorway Span | 41 | 41% |

### 8.3 Shape Types (88 total items)

All shapes are `moistureArea` type (100%).

### 8.4 Freehand Drawing Types (16 total items)

All freehand drawings are `moistureArea` type (100%).

---

## 9. Water Classification

Only 120/212 records (57%) include water classification fields. These were added in a later schema variant.

### Water Category (n=120)

| Category | Count | Share |
| --- | --- | --- |
| CAT 2 (Grey Water) | 54 | 45% |
| CAT 1 (Clean Water) | 44 | 37% |
| CAT 3 (Black Water) | 18 | 15% |
| Not Defined | 4 | 3% |

### Water Class (n=120)

| Class | Count | Share |
| --- | --- | --- |
| Class 2 | 63 | 53% |
| Class 1 | 31 | 26% |
| Class 3 | 21 | 18% |
| Not Defined | 4 | 3% |
| Class 4 | 1 | <1% |

**Note:** All 4 "Not Defined" records belong to a single tester (`019BC706…`) from one session on 2026-01-21, suggesting they skipped the classification step.

---

## 10. Room Scan Complexity (capturedRoom)

Each chamber contains a LiDAR-captured room with walls, floors, doors, windows, openings, and detected objects.

| Component | Chambers with Data | Max Count | Mean Count |
| --- | --- | --- | --- |
| Walls | 209/212 (99%) | 55 | 15.2 |
| Floors | 209/212 (99%) | 1 | 1.0 |
| Objects (furniture) | 192/212 (91%) | 69 | 16.5 |
| Doors | 192/212 (91%) | 13 | 2.6 |
| Windows | 161/212 (76%) | 11 | 2.4 |
| Openings | 124/212 (59%) | 5 | 1.0 |

### Additional capturedRoom Metadata

| Field | Value |
| --- | --- |
| Room scan version | v2 (198/212), v1 (14/212) |
| Story (floor level) | Always `0` (ground level) |
| Active layer | `moisture` (137), `equipment` (51), `floorPlan` (15) |

---

## 11. Chamber Naming

| Chamber Name | Count |
| --- | --- |
| First Drying Chamber | 172 (81%) |
| Chamber 1 | 11 |
| Two bed one bath | 7 |
| Chamber 2 | 7 |
| Entry | 3 |
| Other (9 names) | 12 |

**Key Insight:** 81% of chambers use the default name "First Drying Chamber", indicating most testers don't rename. The few who do rename provide descriptive names (room types, numbered chambers).

---

## 12. JSON Payload Size

| Metric | Value |
| --- | --- |
| Min | 726 chars |
| Max | 48,509 chars |
| Mean | 15,911 chars |
| Median | 12,995 chars |

The largest payload (48.5 KB) comes from a complex room scan with many walls — the `capturedRoom` object dominates payload size. The smallest payloads (< 1 KB) are minimal scans with few walls and no annotations.

---

## 13. Floor Area Sample

A polygon area calculation on a sample floor scan (`First_Drying_Chamber_2026-01-28_121556`) yields:

| Metric | Value |
| --- | --- |
| Area | 93.89 m² (~1,011 sq ft) |
| Perimeter | 39.54 m |
| Valid polygon | Yes |

This demonstrates that floor polygon coordinates from `capturedRoom.floors[].polygonCorners` can be used to compute room areas programmatically.

---

## 14. Key Takeaways

1. **Small but active beta group** — 21 testers with highly skewed engagement; top 3 drive 58% of data.
2. **Equipment is the primary annotation** — 37% adoption, with Air Movers accounting for 67% of placed equipment.
3. **Water classification is inconsistently captured** — Only 57% of records have it; likely a schema addition mid-test.
4. **Room scans are rich** — Average 15 walls, 17 objects, 3 doors per scan; LiDAR data quality appears high.
5. **Default naming is dominant** — 81% "First Drying Chamber" suggests the rename UX may be overlooked.
6. **Moisture tools have low but intensive adoption** — Wall moisture (14%) and shapes (26%) are used deeply when engaged.
7. **Global reach** — 7 countries, 25 subdivisions, consistent with distributed field testing.

---

## 15. Recommendations for Further Analysis

- **Compute room areas at scale** — Use floor polygon coordinates to derive area distributions across all 212 chambers.
- **Correlate feature usage with engagement tier** — Do power users annotate more, or just scan more rooms?
- **Analyze equipment density per room area** — Are Air Mover counts proportional to room size?
- **Track schema evolution** — The 4 key-set variants suggest iterative app updates during the test period.
- **Investigate v1 vs v2 room scans** — 14 records use v1; compare scan quality/complexity.
- **Evaluate wall moisture placement patterns** — Do moisture readings cluster on exterior vs interior walls?

---

*Generated from `primary-analysis.ipynb` — see notebook for full code and visualizations.*
*Chart assets: `assets/chamber_feature_usage.png`, `assets/chamber_value_distribution.png`*
