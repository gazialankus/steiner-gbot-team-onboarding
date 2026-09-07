# SmartSense — Team Onboarding · START HERE

GBot Software · September 7, 2026

Welcome. This bundle is everything you need besides the client's own September package (`SmartSense_Release_1_Handoff_Short_Paths_With_Portal_Preview.zip`, already in the Drive folder). Read this page first; it tells you what the project is, what to read in which order, and the ground rules.

## The project in one paragraph

SmartSense is TPMS + telematics for towable RVs (client: Steiner). BLE tire sensors talk to an ESP32 **Gateway** (LTE/GPS → AWS IoT Core) or, in **Direct TPMS mode**, straight to the owner's phone. We are building the whole platform new: **Serverpod/Dart backend + PostgreSQL + AWS IoT**, a **Jaspr** web portal, and a **Flutter** mobile app with both modes. The legacy Django backend and React portals are abandoned reference material — no user will ever touch them again, and neither will we except to learn from them. The client documents the product through AI-generated controlled documents; the decisions inside are real, the ceremony around them is not, and part of our job is telling those apart — the analysis documents in this bundle do that for you.

## Reading order

1. **`1-Analysis/SEPTEMBER2026-DOCS-REVIEW.md`** — our full review of the September package: what governs, what's stale, what's missing, what carries over. The map to everything else. Read completely.
2. **`1-Analysis/DOMAIN-INVARIANTS-AND-VOCABULARY.md`** — the controlled vocabulary and numbered invariants (v0.1 draft). This is a working document you will help harden. **Use its terms and only its terms** in code, schemas, and discussion; cite invariants by ID in reviews. Open questions live in one place: **`1-Analysis/OPEN-QUESTIONS.md`** — every question has one canonical `OQ-` number there (your B*/N* ids are recorded as aliases); new questions get their number from that file, nowhere else.
3. The client's September package (from the Drive zip): `00_Index` first — everyone; then your lane's doc (`01` frontend / `03` backend / `04`+`05` mobile). Inside its `Portal_Source.zip`, the **Alerts Backend Implementation Specification** is the best-engineered document in the whole handoff — backend devs read it end to end.
4. **`2-Frozen-Authority/SmartSense-Logic-Explorer.html`** — double-click it; it's a self-contained, cross-linked browser for the frozen product authority (72 Merge IDs, decisions, open items). Use it whenever a doc cites an `M`, `D`, or `OD` number. The two spreadsheets next to it are the actual frozen authority the explorer renders (Register v0.10 + Process Spec v0.8) — rank #1 whenever documents disagree.
5. **`3-J3309-Safety/`** — mobile + firmware lanes: the RevD spec is the governing alerts/HMI document (gateway-primary, dual BLE streams, Event-ID dedup, timing targets).
6. **`4-Gateway-Reference/`** — the gateway technical spec and Annexes (MQTT parameter catalog, BLE GATT, alert mapping, OTA). Note: our Annex-1 copy is v2; the client's newest docs cite v4.1, which we've requested.
7. **`5-GBot-Backend/SERVERPOD-ARCHITECTURE.md`** — the backend architecture plan; read the caveat banner at its top.

## Ground rules

- **Never touch Steiner's AWS or any Steiner-owned system without Gazihan's explicit go.** A standing compliance log records every change GBot has ever made there; that discipline continues.
- **Authority order** when documents disagree: frozen Register v0.10 + Spec v0.8 → accepted portal source + its QA → Alerts backend spec → J3309 packet → Annex-1 → everything older is historical. The review doc §3 lists the known stale-document traps (route counts, QA counts, superseded permission matrix) — check it before trusting any number in a client doc.
- **Vocabulary discipline**: one term per concept, banned synonyms stay banned ("alarm", "alert rule", "online/offline" for VINs…). If a concept has no term yet, propose one — don't improvise silently.
- **The system never converts ignorance into a claim**, and **it rolls with the punches** — read principles P-1..P-5 in the invariants doc; they are the product's character.
- Client docs are AI-generated: verify before relying, and flag anything that smells invented. The review doc §8 has the slop filter.

## Team working documents (`6-Team-Working-Docs/`)

Three documents Bilal and Mert produced on September 4, reviewed and endorsed September 7 — they are part of the canon, not sidecar notes:

- **SmartSense Route Register** — all 47 routes with page policy, protected actions, field gates, grid/query contracts, and the S1–S6 build order (34 routes in Release 1). It independently confirmed the 47-routes/235-cells registry truth against the stale 46/230 documents and raised the discrepancy with the client.
- **SmartSense Domain Model** — 51 entities in 11 groups targeting Serverpod/PostgreSQL, with 11 schema invariants, the four missing contracts (AlertEvent store, notification delivery, principal/session, Owner Activation), and fields blocked on Product Owner decisions. Status: **approved by Gazihan 2026-09-07** as the basis for schema work (caveats: opaque production cursors — OQ-37; RV Owners `attention` sort needs a decision — OQ-36; Owner Activation unmodeled until OQ-6). Team blockers B1/B2 (OQ-34) remain. B4 is reshaped (Gazihan, Sept 7): the datapoint catalogue is **general and extensible** — seed it with the prototype's eight measurements (with their Annex-1 gateway-setting mappings), and later datapoints get authored in a governed admin surface (OQ-14). Groups A/B/J are clear to scaffold and group E can test against the seed; client still to bless the extensible model and the new Admin › Datapoints route.
- **SmartSense Portal UX Review** — 26 walkthrough observations dispositioned (17 ours, 6 needing a Product Owner decision, with the ones that collide with prior decisions correctly flagged).

## People

- **Gazihan** — GBot lead; owns client communication and reconciliation of all open questions.
- **Bilal** and **Mert** — GBot developers; authors of the team working documents in `6-Team-Working-Docs/` (Route Register, Domain Model, UX Review) and owners of the `B*`/`N*` question aliases in the register.
- **Tom** — Steiner's product/AWS/mobile counterpart; source of the controlled packages.
- **Dimitar** — Steiner's gateway firmware/hardware engineer; supplied the gateway devkit.

## Code (repo access being set up)

- `smartsense-platform` — the production seed: Serverpod 4 workspace with live AWS IoT ingest, pilot DB, Flutter Web portal, Jaspr port in progress (`JASPR-PORT.md`).
- `Drive-Pressure` — Direct-mode iOS prototype (encrypted sensor BLE, background scan, CarPlay); only written-down packet layout for the current sensor firmware.
- `Mobile-App` — the taken-over Flutter app; overhaul decision pending.
- Legacy `Backend-App` (Django) and `Admin-Portal` (React) — reference only; `Backend-App/docs/Gateway_proto.csv` is the parameter-ID source of truth until reseeded.

Questions → Gazihan. Proposed corrections to the analysis or invariants docs → PR/comment, don't edit the frozen client documents themselves.
