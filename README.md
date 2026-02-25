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
|-----------|-------------|
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

```
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
└── 05-combined-json-wip/                  ← Combined JSON from Anton (Feb 24)
    ├── README.md
    ├── input-contract-spec.md             ← Formal input contract spec (Draft v0.1)
    ├── 24-feb-Untitled.json               ← mainTour + instantSketch flat payload (WIP)
```

---

## Current Status (as of Feb 25, 2026)

### Team BE → Team Core (Input Contract)

| Field / Feature | Status | Date Added |
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
| Treatment info/material | Blocked (mobile app) — Ahmed asked Oleg Shultsev | — |
| Wall moisture readings | Empty in current file | — |
| Saturation level | Not present | — |
| Dehumidifier type (LGR vs conventional) | Not present | — |

### Team Core → Team BE (Output Contract)

In discussion, with primary alignment on ownership of fields.

### Transport

| Detail | Value |
| --- | --- |
| Broker | Amazon SQS (N. Virginia) |
| Queues | 2 — input queue + return queue |
| Size limit | 256KB (S3 fallback if exceeded) |
| Flow | Async: Team BE → input queue → Team Core processes → return queue |
| DLQ / retries | Deferred until formats finalized |

---

## Open Dependencies

| Item | Owner | Blocked By |
| --- | --- | --- |
| Combined JSON structure | Anton (Team BE) | **Resolved** — delivered Feb 24 (WIP) |
| Treatment info/material | Oleg Shultsev (Mobile) | Mobile app feature — Ahmed asked Oleg Shultsev |
| Mandatory fields in Instant Sketch app | Oleg Shultsev (Mobile) | Malte requests all except moisture readings be mandatory |
| Wall moisture readings | Anton (Team BE) | Empty in current file — depends on mobile app |
| Saturation level | — | **Resolved** — derivable from `wallMoisture` readings once populated |
| Dehumidifier type (LGR vs conventional) | Ahmed → Anton (Team BE) | Not in current schema |
| AWS SQS access | Ahmed | **Resolved** |
| SQS queue names, DLQs, retries, timeouts | Ahmed + Anton | Deferred until format stable |
| Output contract definition | Ahmed + Mladen | Input contract finalization |
| Validation & failure behavior | Ahmed + Anton | Schema stabilization |
| Versioning strategy | Both teams | — |

---

## Teams & Contacts

| Team | Role | Key Contact |
| --- | --- | --- |
| **Team iOS** | Instant Sketch iOS | Oleg Shultsev |
| **Team BE** | Backend / Converter (Instant Sketch → FE JSON) | Oleg D. & Anton K. |
| **Team Core** | Core Team (line item creation) | Malte K. & Ahmed S. |
| **Infra** | AWS access | Martin, Akash |
