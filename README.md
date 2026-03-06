# PEP Data Contracts

> Define the initial dependencies and interface contracts between Team A, Team B, and Team C for Q1.

- **Miro Board:** [Interface Contracts Board](https://miro.com/app/board/uXjVGQtrS10=/)
- **JIRA:** [PEP-12 — Feasibility & Risk Assessment](https://clipnow.atlassian.net/browse/PEP-12)

---

## Purpose

Make handoffs explicit — data formats, responsibilities, ownership boundaries — and provide a concrete basis for integration planning, contract tests, and feasibility/risk assessment.

---

## Scope

- Identify all Q1-critical handoffs between Team A ↔ Team B ↔ Team C.
- Define responsibilities and "source of truth" boundaries for each handoff.
- Specify payload contracts for each interface (schemas, required fields, optional fields, versioning).
- Define evidence and provenance requirements for handoff payloads (e.g., evidence pointers, timestamps, confidence where applicable).
- Define validation and failure behavior at boundaries (reject vs accept-with-warnings, unknown handling, backward compatibility expectations).
- Capture open dependencies and sequencing constraints relevant for the Q1 integration plan.

---

## Deliverables

### Interface Contract Specifications

| Interface | Description |
| --- | --- |
| **Team A → Team B** | Normalized text + evidence references + minimal context |
| **Team B → Team C** | Canonical context + conflicts + required summaries |

### Supporting Artifacts

- Responsibility matrix covering ownership of transformations and sources of truth.
- Initial dependency list with owners and expected delivery order.
- Contract test plan outline (what will be validated automatically at boundaries).

---

## Acceptance Criteria

- [ ] Each Q1-critical handoff has an explicit owner, payload contract, and validation rules.
- [ ] Contracts define required fields, provenance requirements, and a versioning strategy.
- [ ] Failure and unknown-handling behavior is documented for each interface.
- [ ] Contracts and dependencies are reviewed with Team A and Team C and recorded as the Q1 baseline.
- [ ] All identified dependencies and interface assumptions are reviewed as explicit inputs to the Q1 feasibility and risk assessment ([PEP-12](https://clipnow.atlassian.net/browse/PEP-12)).
- [ ] Miro Board reflects these contracts and dependencies.

---

## Repository Structure

```plaintext
Data Contracts/
├── README.md                              ← You are here
├── TODO.md                                ← Open work items & missing JIRA deliverables
├── 01-raw-ios-data-analysis/              ← Raw iOS data analysis (Mixpanel export, 212 chambers)
│   ├── README.md
│   ├── data-report.md                     ← TestFlight data export analysis
│   ├── primary-analysis.ipynb             ← Jupyter notebook with full analysis
│   ├── chamber_feature_usage.png
│   ├── chamber_value_distribution.png
├── 02-initial-fe-contract/                ← First FE.json (Feb 4)
│   ├── README.md
│   ├── FE_full.json                       ← Initial FloorplanEditorSketch JSON
│   └── floorplan_editor.md                ← Full FE JSON format specification
├── 03-aligned-fe-contract/                ← Second FE.json (Feb 6–12)
│   ├── README.md
│   ├── FE.json                            ← + roomId, projectData, moistureArea type, wall length
├── 04-combined-json-prototype/            ← Combined JSON prototype (Feb 16)
│   ├── README.md
│   ├── Untitled.json                      ← mainTour + panos + maps (combined structure WIP)
├── 05-combined-json-wip/                  ← Combined JSON from Anton (Feb 24)
│   ├── README.md
│   ├── input-contract-spec.md             ← Formal input contract spec (Draft v0.1)
│   ├── 24-feb-Untitled.json               ← mainTour + instantSketch flat payload (WIP)
└── 06-mar-status-update/                  ← Status update & new data inventory (Feb 26 – Mar 4)
    ├── README.md                          ← Full narrative of what moved since Feb 25
```

---

## Current Status (as of Mar 5, 2026)

### Team BE → Team Core (Input Contract)

**Delivered:**

| Field / Feature | Status | Date |
| --- | --- | --- |
| Walls, corners, doors, windows, holes | Delivered | Feb 4 |
| Equipment (with `roomId`) | Delivered | Feb 6 |
| `projectData.waterCategory` / `waterClass` | Delivered | Feb 6 |
| Shapes with `type: "moistureArea"` + `affectedSurfaces` | Delivered | Feb 6 |
| Wall moisture (by `wallId`) | Delivered | Feb 4 |
| Containment barriers (`type: "barrier"`) | Delivered | Feb 4 |
| Wall `length` | Delivered | Feb 9 |
| Enum normalization (capitalized, arrays) | Delivered | Feb 12 |
| Combined JSON (FE.json + DocuSketch project data) | Delivered (WIP) | Feb 24 |
| `mainTour` with panos, attachments, `typeOfLoss` | Delivered | Feb 24 |
| `instantSketch` with rooms, walls, equipment, moistureAreas, barriers | Delivered | Feb 24 |
| Direct `roomId` linkage (pano → room, replacing `maps`) | Delivered | Feb 24 |
| Full Core `/line-items` endpoint schema shared with BE | Delivered — sandbox URL, docs, POST `/line-items` with curl examples | Mar 4 |
| Material types (16 from Encircle) | Delivered by Valeriya — Carpet, Drywall, Concrete, Wood Sill Plate, Ceramic Tile, Hardwood, Laminate, Plywood, Vinyl, Plaster, Paneling, Subfloor, Framing, Particle Board, Gypcrete, OSB | Mar 3 |
| Equipment types JSON (Dehumidifier, Air Mover, Air Scrubber) | Delivered by Valeriya — includes pints-per-day rating (sufficient to distinguish LGR vs conventional) | Mar 3 |
| Wall moisture data structure | Clarified by Kirill — measurement values (float % X.X, float points X.X), inside/outside air readings (Temp °F, RH %, Humidity Ratio gpp), dew points & vapor pressure (calculated) | Feb 26 |
| `is_business_hours` definition | Resolved — Chris Tilkov confirmed standard insurance definition: after-hours = outside 5pm–8am, weekends, statutory holidays. Deterministic from creation-time. No per-contractor config needed | Mar 2 |
| Missing fields list approved by Malte | `floor_type` (enum), `is_floor_salvageable` (bool), `wall.material_type` (enum), `job.call_timestamp` (datetime), `job.emergency_call` (bool). Malte: prefill `is_business_hours` from date, let user override | Mar 2 |

**Blocked / Pushback:**

| Field / Feature | Status | Date |
| --- | --- | --- |
| New field requests — Valeriya pushback | 6 fields posted to #instant_sketch_public (`floor_type`, `is_floor_salvageable`, `wall.material_type`, `wall.moisture_height_inches`, `job.is_business_hours`, `job.emergency_call`) — NOT in current implementation. Valeriya: "a lot of manual input fields for the user." **Alignment call needed — not yet scheduled** | Mar 3 |
| Treatment info/material | Not available — Valeriya confirmed only material types exist, no treatment info in system | Mar 3 |
| Wall moisture readings | `wallMoisture[]` data structure clarified by Valeriya but still depends on mobile app implementation. Production-ready solution TBD | Feb 26 |
| Saturation level | Derivable from `wallMoisture` once populated — unchanged | — |
| Lines/dots duplication | Not discussed since Feb 25. Status unknown | — |
| Dehumidifier type (LGR vs conventional) | Potentially resolvable — equipment JSON includes pints-per-day rating, but not yet confirmed as sufficient | Mar 3 |

### Team Core → Team BE (Output Contract)

Partial progress. Ownership on some fields agreed with Mladen & Oleg, but schema is not finalized. BE team (Anton + Eran) are eager to consume Core output — their own LLM-based line item extraction is producing poor quality (missing items during ESX conversion). Alignment with Malte scheduled for Mar 5 to progress this.

### Transport

| Detail | Value |
| --- | --- |
| Mechanism | **S3-based delivery** (agreed Mar 4 with Anton) |
| Flow | BE saves FE.json to S3 → sends `s3_uri` in evidence payload → Core fetches and processes. No inline JSON needed — S3 path reference only |
| Return | TBD — return queue or callback not yet defined |
| SQS | Originally planned, now deferred. May still be used for return path |
| DLQ / retries | Deferred until formats finalized |
| Queue counterpart | Shifting from Anton → **Mladen Djordjevic** (Anton moving to IS mobile) — to be confirmed Mar 5 |

### BE Team Urgency (New since Feb 25)

Anton's team and Eran are actively pressing to consume Core output. Their own LLM-based line item extraction is producing poor quality — items go missing during ESX conversion. They want to test Core results as an alternative. Ahmed shared sandbox access (`ste-sandbox.v2docusketch.com`) and full endpoint documentation on Mar 4.

### Additional Context

- **Chris Tilkov (Mar 2–4):** Business hours confirmation (above). Also provided video feedback on Andrew Wirick's line item mobile prototype in #preestimate_core_team — how users should interact with line items after Core populates them. Relevant to output/UI (Team C territory).
- **Kirill (Feb 26):** Moisture and air readings are two separate data types — Core needs both. Moisture data exists in Prelim Estimate MS for prototype only. Production-ready TBD. Also working on on-device moisture meter reading recognition (Gemini Pro ~90% accuracy, exploring native Apple Vision framework — mixed results). Directed Ahmed to Drago Jeremic for current data structure.
- **Malte (Mar 2–3):** Pushing for validation dataset and evaluation platform to track metrics. Suggested simulating actual input format from past estimates even before facts are finalized. Sprint review Thursday is the priority — video update not needed. 1:1 rescheduled to Thursday (Ahmed's Ramadan schedule)

---

## Open Dependencies

| Item | Owner | Status (Mar 5) |
| --- | --- | --- |
| Combined JSON structure | Anton (Team BE) | **Resolved** — delivered Feb 24 (WIP) |
| S3-based transport for FE.json | Anton → Mladen (Team BE) | **Agreed** Mar 4 — BE saves to S3, sends `s3_uri` |
| Full Core endpoint schema shared | Ahmed | **Resolved** — sandbox + docs shared Mar 4 |
| Material types inventory | Valeriya | **Resolved** — 16 types delivered Mar 3 |
| Equipment types inventory | Valeriya | **Resolved** — JSON with pints-per-day delivered Mar 3 |
| `is_business_hours` definition | Chris Tilkov | **Resolved** — standard insurance definition, deterministic from timestamp |
| Missing fields approved by Malte | Malte | **Resolved** — 5 fields approved Mar 2 |
| AWS SQS access | Ahmed | **Resolved** |
| Saturation level | — | **Resolved** — derivable from `wallMoisture` readings once populated |
| Alignment call with Valeriya re: new fields | Ahmed + Valeriya | **Blocked** — Valeriya pushed back, call not yet scheduled |
| Treatment info | Valeriya | **Not available** — only material types exist in system |
| Mandatory fields in Instant Sketch app | Oleg Shultsev (Mobile) | Malte requests all except moisture readings be mandatory — no update |
| Wall moisture readings | Drago Jeremic / Mobile | Data structure clarified, production-ready solution TBD |
| Dehumidifier type (LGR vs conventional) | Ahmed → Anton | Equipment JSON has pints-per-day — may resolve this, needs confirmation |
| Lines/dots duplication | Anton (Team BE) | Not discussed since Feb 25 |
| Output contract definition | Ahmed + Malte | Partial progress — alignment scheduled Mar 5 |
| SQS queue names, DLQs, retries, timeouts | Ahmed + Mladen | Deferred — Anton handing off to Mladen |
| Validation & failure behavior | Ahmed + Mladen | Schema stabilization |
| Versioning strategy | Both teams | — |
| Validation dataset + evaluation platform | Malte | Malte pushing for this |
| On-device moisture meter recognition | Kirill | Gemini Pro ~90% accuracy; Apple Vision mixed results |

---

## Teams & Contacts

| Team | Role | Key Contact |
| --- | --- | --- |
| **Team iOS** | Instant Sketch iOS | Oleg Shultsev |
| **Team IS Data** | IS data inventory & field definitions | Valeriya, Kirill |
| **Team BE** | Backend / Converter (Instant Sketch → FE JSON) | Oleg D., Anton K. (→ moving to IS mobile), **Mladen Djordjevic** (new queue counterpart) |
| **Team Core** | Core Team (line item creation) | Malte K. & Ahmed S. |
| **Product / Stakeholders** | Product direction, insurance domain | Chris Tilkov, Eran |
| **Moisture Data** | Current moisture data structure | Drago Jeremic (directed by Kirill) |
| **Infra** | AWS access | Martin, Akash |
