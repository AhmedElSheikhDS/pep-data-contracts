# 06 — Status Update (Feb 26 – Mar 4)

- **Date range:** February 26 – March 4, 2026
- **Sources:** Slack conversations with Anton, Valeriya, Kirill, Malte, Chris Tilkov

---

## What Moved Since Feb 25

### Anton (Mar 4)

- **BE team urgency:** Anton's team + Eran are pressing for Core integration. Their own LLM-based line item extraction is producing poor quality — missing items when converting to ESX. They want to consume Core output instead.
- **Sandbox shared:** I (Ahmed) shared sandbox URL (`ste-sandbox.v2docusketch.com`), docs, and full input/output schema for `POST /line-items` endpoint with curl examples.
- **Transport agreed:** BE saves the FE.json to S3, sends the `s3_uri` in the evidence payload. Core fetches and processes. No inline JSON needed. Anton confirmed and accepted.
- **Ownership change:** Anton is moving to help mobile devs with InstantSketch. **Mladen Djordjevic** will likely be Ahmed's counterpart for queue setup going forward.

**Not discussed:** Dehumidifier type, lines/dots duplication, SQS queue names/DLQs/retries.

---

### Valeriya (Feb 26 – Mar 3)

**Feb 26:** Acknowledged data request, said she'd provide the list in sync with Kirill.

**Mar 3 — Data inventory delivered:**

#### Wall Moisture

- Measurement values: float % (X.X), float points (X.X)
- Inside/outside air readings: Temperature (°F), Relative Humidity (%), Humidity Ratio (gpp)
- Dew points & vapor pressure (calculated)

#### Equipment Types

- Shared as attached JSON
- Models: Dehumidifier, Air Mover, Air Scrubber (sourced from Encircle)
- Includes pints-per-day rating — sufficient to distinguish LGR vs conventional dehumidifiers

#### Material Types (16)

Carpet, Drywall, Concrete, Wood Sill Plate, Ceramic Tile, Hardwood, Laminate, Plywood, Vinyl, Plaster, Paneling, Subfloor, Framing, Particle Board, Gypcrete, OSB

#### Treatment Info

- **Not available** — only material types exist in the system

**Mar 3 — Pushback on new fields:**

I (Ahmed) posted 6 new field requests to #instant_sketch_public:

1. `floor_type` (enum) > only 2 material types are not caputured
2. `is_floor_salvageable` (bool)
3. `wall.material_type` (enum)
4. `wall.moisture_height_inches` (float)
5. `job.is_business_hours` (bool) > can be fetched from IS app
6. `job.emergency_call` (bool)

Valeriya flagged these are NOT in current implementation: "a lot of manual input fields for the user." Wants an alignment call before committing. **Call not yet scheduled — this is the main blocker on input completeness.**

---

### Kirill (Feb 26)

- Clarified moisture vs. air readings are two different data types — Core needs both for the full picture.
- Moisture data exists in Prelim Estimate MS for prototype, but production-ready solution is TBD.
- Directed Ahmed to **Drago Jeremic** for current moisture data structure.
- Separately working on on-device moisture meter reading recognition:
  - Gemini Pro: ~90% accuracy
  - Apple Vision framework: mixed results

---

### Malte (Mar 2–3)

- **Approved missing fields list:** `floor_type`, `is_floor_salvageable`, `wall.material_type`, `job.call_timestamp`, `job.emergency_call` — "everything else makes sense."
- **Challenged `call_timestamp`:** Suggested boolean flag instead. Ahmed escalated to Chris Tilkov, who confirmed the standard insurance industry definition for after-hours (outside 5pm–8am, weekends, statutory holidays). `is_business_hours` is deterministically computable from creation-time.
- **Malte's final position on business hours:** Prefill based on date, let user override.
- Meeting rescheduled to Thursday (Ahmed's Ramadan schedule). Sprint review Thursday is the priority — video update not needed.
- Pushing for **validation dataset** and **evaluation platform** to track metrics. Suggested simulating actual input format from past estimates even before facts are finalized.

---

### Chris Tilkov (Mar 2–4)

- Confirmed business hours definition (see Malte section above).
- Mar 4: Provided video feedback on Andrew Wirick's line item mobile prototype in #preestimate_core_team — suggestions on how users should interact with line items after Core populates them. Relevant to output/UI (Team C territory) but shows active engagement on end-user experience.

---

## Summary Table

| Area | Feb 25 Status | Current Status (Mar 5) |
| --- | --- | --- |
| Input transport | SQS planned but not configured | S3-based delivery agreed with Anton. SQS deferred |
| Input schema | Combined JSON delivered as WIP | Full Core endpoint schema shared with BE team. BE eager to consume |
| Missing fields | Identified but not requested | Requested formally, Malte approved. Valeriya pushed back — alignment call needed |
| Material types | Blocked on mobile | 16 material types provided by Valeriya (from Encircle) |
| Equipment types | Not in schema | Equipment JSON delivered with pints-per-day rating |
| Wall moisture | Empty in file | Data structure clarified (%, points, air readings). Production-ready TBD |
| Business hours | Unresolved | Resolved. Standard insurance definition confirmed by Chris Tilkov. Deterministic from timestamp |
| Dehumidifier type | Not in schema | Potentially resolvable via pints-per-day — needs confirmation |
| Treatment info | Blocked on mobile | Not available. Only material types exist |
| Output contract | In discussion | Partial progress. Alignment with Malte scheduled Mar 5 |
| Queue counterpart | Anton | Shifting to Mladen (Anton moving to IS mobile) |
| BE team urgency | — | High. LLM extraction quality poor, want Core output |
| Validation platform | — | Malte pushing for dataset + eval metrics |

---

## Key Actions (Mar 5)

1. Align with Malte on output contract schema finalization
2. Confirm with Mladen he's taking over queue work from Anton
3. Schedule alignment call with Valeriya re: missing input fields — main blocker on input completeness
