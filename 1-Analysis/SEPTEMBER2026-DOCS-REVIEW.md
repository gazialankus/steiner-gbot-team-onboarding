# September 2026 Handoff — Review and Team Preparation Notes

GBot Software · working notes · September 7, 2026
Scope: `Docs/September2026/SmartSense_Release_1_Handoff_Short_Paths_With_Portal_Preview` (six docx dated Sept 3, portal preview, `Portal_Source.zip`), checked against the frozen August baseline, our own decisions, and our existing code.

---

## 1. What arrived

- **Six handoff documents** (Sept 3, 2026): `00_Index` (baseline, authority order, gates), `01_Frontend_Spec` (shell, routes, tenant, access matrix), `02_Alerts_Handoff` (alerts module), `03_Serverpod_Handoff` (backend contract), `04_Notifications_Mobile`, `05_J3309_Handoff`.
- **`Portal_Source.zip`** → `_handoff/SmartSense_QE_PreChange_Review/` with three parts: `01_Portal_As_Accepted` (the accepted prototype + its manifests, QA, permission model, the Alerts Backend Implementation Specification, the contract spine JSON), `02_Review_Documents` (~40 design-review notes, July 31 – Aug 31), `03_Standing_Instructions`.
- **Portal preview** (`SmartSense_Portal.dc.html` + `support.js` + `geo-map.jsx`) at package top level.

**Verified with hashes (not taken on trust):**

- The authoritative portal source is `01_Portal_As_Accepted/SmartSense Portal.dc.html`, SHA-256 `24aa9dfe…806f1a` — matches the index's claim exactly.
- The top-level preview is a **standalone export** of that same source (SHA `e98326a7…`, byte-identical to `01_Portal_As_Accepted/standalone/`; diff is only bundler metadata). Don't treat the preview file as the hash-authoritative artifact.
- All 43 hashes in `MANIFEST - SHA256.md` verify byte-exact. Their hash-chain process actually works. Seven files are unmanifested, including the two most recent reviews (`Pre-Demo UI Review`, `Screen by Screen Review`, both Sept 2 — see §8/§10).

## 2. Headline: our stack is now the client's own decision

`03_Serverpod_Handoff` §1: *"The production stack is Serverpod/Dart with PostgreSQL, AWS-hosted services, and a Jaspr-based Portal frontend. Flutter Mobile Apps remain separate clients."*

The August framework-gate conflict (Serverpod-likely vs stays-on-Django) is closed in our favor, in writing, in the client's own controlled documents. The same document states no Serverpod project exists yet — greenfield is not just permitted, it's the documented plan. The frozen Register's platform-neutrality principle is superseded on this point.

## 3. Authority order and stale-document traps

`00_Index` §3 ranks the sources. When documents disagree:

1. **Frozen product authority: Critical Logic Register v0.10 + Process Specification v0.8** (the August package spreadsheets — still #1).
2. Current Portal source (`24aa9dfe…`) + its QA results (runner 55.0.0, 1,659 assertions, prototype-only evidence).
3. Alerts Backend Implementation Specification (+ approved alerts decision record).
4. J3309 transfer packet, clause matrix, addendum.
5. Annex-1 Raw Gateway Data Model workbook (names/units/setting IDs only — never threshold values).
6. Older reconciliation reports, portal reviews, handoff READMEs: **historical evidence only**.

The package itself warns that stale documents linger inside it. We verified the specific traps:

| Stale claim | Where it survives | Actual truth |
|---|---|---|
| 46 routes / 230 policy cells | `Permission Model - Release 1.md`, `Workflow Reconciliation`, `01_Frontend_Spec` | Runtime registry: **47 routes / 235 cells** (post-reset Alerts rebuild) |
| 1,603 QA assertions ("any other count is stale") | Backend spec §0.2, `QA SCOPE…1603…md` | `qa/results.json`: **1,659**, runner 55.0.0, 0 failures |
| Portal baseline `2ef728fa…` | Backend spec status block | Accepted source is `24aa9dfe…` (six wording cycles later; spec content unaffected) |
| 11 capability gates | `01_Frontend_Spec` §5 | Portal source carries **12** — `distribution.deploy` (production deployment control) is missing from the frontend spec |
| Granular `domain.object.action` capabilities | `Master SmartSense Portal Access Matrix.dc.html` (Rev 2, Aug 18) | Superseded Aug 22: **five-profile model**, 40 granular capabilities retired; authority is `Permission Model - Release 1.md` + runtime registry |
| Alert Rules / Notifications admin as routes | `Workflow Reconciliation` route table | Deleted in the Aug 31 Product-Owner controlled reset |

Rule of thumb for the team: **trust the runtime registry, `qa/results.json`, and the latest-dated ledger sections over any prose document.**

*Corroboration (Sept 7):* the team's own **Route Register** (Sept 4, extracted from the prototype's `portalRoutes()`) independently found the same 47-routes/235-cells truth, explained the drift (the clean Alerts module retired three route rows and minted four), and has **already raised it with the client in the handoff review** — so that trap is both confirmed and reported.

## 4. Check against our technical decisions

**a) Serverpod + Jaspr, whole new backend, never touches the legacy backend.**
Confirmed by the docs (§2 above). Independence also matches our standing rule in `CHANGES-TO-STEINER-SYSTEMS.md` (no changes to their AWS without green light). **Decided (Gazihan, Sept 7): the legacy Django backend serves no users and will be abandoned — kept strictly as an engineering reference.** That makes the August package's four "APPROVED NOW" PRE-B0 Django security patches moot; we do not fix anything in the legacy code. Worth one confirming sentence with Tom so nobody on their side believes those patches are still owed.

**b) AWS IoT: identify the good existing engineering, build a new one inspired by it.**
The docs assume AWS-hosted services but say nothing concrete about IoT ingestion — the good engineering is already identified and documented on our side: `SERVERPOD-ARCHITECTURE.md` §"immovable" (IoT Core, fleet provisioning, per-device X.509, `devices/{serial}/tx|rx` MQTT topics, flat `{parameterID: value}` delta JSON, param 257 = device UTC epoch, `*SET…$` downlink, OTA chunked HTTP, Annex-1 catalog), and `Code/smartsense-platform` already ingests real fleet telemetry from dev AWS IoT through exactly that contract. No conflict; our plan is ahead of their docs here.

**c) Mobile overhaul; direct mode (phone → AWS IoT) + gateway mode; opportunistic upload when gateway LTE is broken.**
- Overhaul/from-scratch: no conflict. The RevD spec (Aug 4) is the governing mobile/HMI document and is architecture-agnostic about the app's insides.
- **Direct mode**: compatible with the frozen Register (Product Mode on the Model is baseline truth) but the September package gives it almost nothing — `05_J3309` explicitly parks it ("a separate applicability and evidence decision"), and OD-009 (direct-mode claim criterion) is still open. Our `Code/Drive-Pressure` prototype is currently the most concrete direct-mode artifact anyone has.
- **Opportunistic upload**: appears in **no client document**. It does not conflict with the safety-layering rule (that rule forbids cloud/push from becoming the *local warning* mechanism; telemetry uplink is a different plane). And RevD's design actually enables it: the gateway already emits a routine BLE telemetry stream (separate GATT characteristic from safety events) and stamps each safety event with a **single Event ID reused across BLE and LTE for dedup** — exactly the primitive an opportunistic phone-uplink needs to avoid double-ingesting. But it is *our invention*: it needs a client decision, plus answers on BLE telemetry completeness vs. the LTE payload, M13 delta semantics for relayed data, M20 event-time preservation, and OD-017 timestamp trust for phone-relayed records. Propose it explicitly; don't build it silently.
- **Watch one reversal**: Rev C→Rev D of the RevD spec **demoted the phone from secondary evaluator to HMI-only** (the app-side independent threshold cross-check became optional diagnostics). In *gateway mode* the phone must not re-evaluate thresholds. In *direct mode* the phone app **is** the evaluator (Register: gateway/phone as sole local evaluator per mode). Keep these two roles from bleeding into each other in the shared codebase.

**d) Background alerts, best effort — the opportunistic-resilience principle.**
Gazihan's framing (Sept 7): "best effort" and "opportunistic" are the same philosophy — *the system does its best to reach its goals despite problems in components, and never uses a component failure as an excuse not to deliver. It rolls with the punches.* Backend down? Gateway/TPMS and phone are right there and warn locally. Gateway LTE dead? The phone can uplink. This is consistent with the docs' safety-layering rules (which forbid *dependence* on cloud, not extra paths) and is now a named design principle in `DOMAIN-INVARIANTS-AND-VOCABULARY.md` (P-2), paired with its guardrail: opportunism never fabricates — degraded paths reduce latency and coverage, never honesty.
One compliance caveat stands: J3309 evidence row GM-J3309-03 requires the Mobile HMI to present gateway warnings **with the cloud path down** — over BLE, including realistic background scenarios — and the RevD timings (detection→BLE ≤250 ms, BLE→presentation ≤250 ms, connection-loss warning ≤1 s) are measured targets. iOS background BLE is the known hard part (the old "iOS 26 Live Activity background scanning" claim was hallucinated; `Drive-Pressure` has real background-scan experience). Define and get client sign-off on what is *claimed* for foreground vs background before evidence collection starts, so best-effort has a written boundary — the engineering keeps rolling with the punches either way.

## 5. Internal contradictions and defects to resolve before/while implementing

Beyond the stale-doc traps in §3:

1. **`alert_definition` schema bug**: `id text PRIMARY KEY` but "unique per organization" (`AD-0001` per org). Needs a composite key or global surrogate. Decide at schema design; don't copy the spec literally.
2. **Envelope conventions**: the alerts spec's `AlertConfigEnvelope` codes (OK/CREATED/…/VERSION_CONFLICT/STORAGE_UNAVAILABLE) differ from the contract spine's `ContractEnvelope` enum (Success/Forbidden/Validation/…), and the spec says no change is required. Pick one convention deliberately for the new backend. (The alerts envelope also retains a `RECORDED` code whose only producer was deleted.)
3. **VIN Detail history end-state is contradictory**: after five redesign reversals the accepted state is "keep the current table, narrow the widths" (PO direction L4) while the PO simultaneously dislikes it (PO-008) and "Candidate B — three evidence lenses" is approved-but-unbuilt. For the Jaspr portal, treat current table = Release 1, Candidate B = the earmarked successor; confirm with Tom.
4. **Significant-G default sort** never formally closed (highest-G-first approved in Phase 1C vs newest-first built and implied by the L2 twelve-month From/To direction). Confirm; newest-first is the likely intent.
5. **Fleet columns, open decision #12**: implemented 8 columns vs the controlled Merge-46 set (Model Year, Status Age, Mileage, Location missing) — flagged in their own audit as an undocumented substitution. Needs a PO decision.
6. **Per-tire SSP on RV Models (Aug 30 programme) vs shared TPMS thresholds across all positions (no per-tire alert definitions)**: different objects (Model spec vs alert threshold) and probably intended to coexist, but no document says so. Confirm.
7. **"Fixed recipient classes" (six docs) understates the accepted model**: releases AA–AC made the recipient an **account** (globally unique id per event+recipient; ack keyed by event id + account; ordinal-based identity withholding "Authorized User 1/2"). Build per-account, not per-class.
8. **Client-timestamp rules read as contradictory but aren't**: `03_Serverpod` "client-supplied timestamps are not trusted" governs the *config-mutation* plane; M10/M20 "preserve source event time + cloud receive time, never overwrite newer with older" governs the *telemetry* plane. State both in the invariants doc so nobody "fixes" one with the other.
9. **The Sept 2 advisory reviews propose things the governed model forbids**: portal-side acknowledge on the Alerts list (accepted model: portal is evidence/read-only; ack is per-recipient, mobile), "assign to a service queue" (the Service-queue recipient class was explicitly deleted in Release X), one unified severity ramp (severity was deliberately decoupled from notification behavior). Use those reviews as UX inspiration only, filtered through the register.

## 6. What the new docs do NOT cover

The six docs are a narrow Release-1 vertical: portal shell → alerts → Serverpod alerts backend → notifications → J3309 evidence. Everything else lives **only** in the frozen Register (authority #1) and is at risk of being forgotten, not superseded:

- **Claim/registration flows** (M1–M5, D-104/105/109/110/114/117, OD-009): zero coverage, and the new development order has **no step for owner claim, gateway binding, replacement, or account recovery** — P0 security territory (HR-01 was a live claim-endpoint hole).
- **Product Mode capability split** (M72, D-121, D-108): the frontend spec's pages will render Stale/Offline/Not Available states but never defines the applicability rules.
- **Telemetry semantics**: delta ingest (M13), backfill (M17/D-106), staleness definition (M16 — the frontend demands a "stale" UI state without defining it), timestamp trust (M10/M20, OD-017).
- **Config drift** (M58 desired/delivered/applied/reported), **health roll-up** (M44–M46), **geofence evaluation logic** (M63–M65), **subscriptions backend model** (M52–M56: VIN snapshot terms, Essentials floor — portal routes deferred, backend truth untouched), **firmware/OTA lifecycle** (M59–M61 — only the old *screens* were rejected, not the logic), **notification breadth** (M36 fallback, M42 escalation, M43 quiet hours/always-on — M43 is a declared unimplemented gap), **mobile flavors** (OD-033).
- **~28 open OD items** still have no landing place; the new docs settle only OD-025 (per-recipient ack/snooze) and give OD-016/M19 a home (the DatapointDefinition service step).
- **Genuine silences needing client answers**: the four PRE-B0 Django patches (§4a); which document governs the *non-alert* backend ("the approved backend specification" in `03` covers alerts only); the Alert Definition production content (RR-002 — reshaped 2026-09-07: authored in the admin panel, no upfront dataset; only the per-Model TPMS safety values remain owed by the client — OQ-1); Owner Activation as an independent fact (currently derived from claim in fixtures; M8 says independent — needs backend definition + mobile contract).

## 7. Do we still need the old docs? Yes — the register is still authority #1

**Required-reading set (keep at hand):**
1. Critical Logic Register v0.10 + Process Spec v0.8 (frozen authority #1)
2. `Docs/SmartSense-Logic-Explorer.html` — the readable cross-linked form of #1
3. The six September docs (authorities #2–#4)
4. `01_Portal_As_Accepted/`: **Alerts Backend Implementation Specification** (the best-engineered document in the whole handoff — our Serverpod contract source), `Permission Model - Release 1.md`, `contracts/portal-contract-spine.v0.6.0.json`, the accepted portal source
5. Annex-1 Gateway Database workbook (authority #5)
6. `04_TPMS_J3309_Safety/` specs incl. **RevD** + traceability matrix (firmware/mobile/compliance lanes)
7. Ownership/Interface Matrix v1.8 (IC-001–012, cited throughout the Register)

**Archive with a "historical/superseded" label:** Reconciliation v0.8 (REC/HR — audited the abandoned Django code; its target-behavior column is still good reading), portal frontend blueprint v0.12, handoff guide v1.1, master index v2.7, old READMEs, old portal/Jungle reviews, Django/mobile/DB evidence zips, the August Prototype Portal package (its successor is the accepted `24aa9dfe…` source). **Confirm disposition before archiving:** Backend AWS Blueprint v0.8 and Mobile Blueprint v0.6 (neither ranked nor superseded by the new order).

## 8. Overkill / slop filter (what to ignore, what's real)

**Ignore / discount:**
- Assertion-count theater: 1,659 "assertions" are fine-grained points in one self-testing JS file (wording changes bump the count). Treat as a few hundred behaviors of *prototype-only* evidence — which the docs themselves admit.
- The four ~3.4 MB "review bundles": forensic concatenations of the same portal HTML; zero design content beyond manifests.
- Release-letter churn (T→AF: thirteen "releases" in two days) and the hash ceremony on wording-only changes. The confessions inside are useful; the process weight is not ours to inherit.
- `TelemetryIngestService` in the portal contract spine: a prototype seam. Telemetry ingest belongs in the device pipeline, never a portal-facing endpoint.
- Test rows 24/25 "prove no UPDATE/DELETE ever reaches a version table" — implement as grants/policy, not as an executable test as written.
- **Apple Critical alerts in R1**: gated everywhere, but nobody flags the entitlement dependency. *(Revised 2026-09-08: no longer post-R1 — Sensata's AirCheck BLE app has the entitlement and delivers Critical Alerts from the background, and Gazihan set AirCheck parity as our floor (OQ-4). Start the entitlement application through the shipping Apple developer account now — OQ-43.)*
- Every "verified" claim in the review notes: the archive itself documents repeated false-completeness (features asserted that the build lacked). Reproduce before relying.

**Genuinely good engineering to adopt:**
- The atomic `disableAndClose` transaction design (full annotated SQL, idempotent retries, momentary events excluded), append-only version tables, and the CHECK-constraint defense against a configuration closure impersonating a Gateway Clear.
- "The request never carries the population" (WindowedQuery), non-disclosing NOT_FOUND for out-of-scope rows, denials-audited-too.
- The base-profile-ceiling custom-profile model.
- The reporting-status pivot (Releases AD–AF): Online/Offline is dead; server resolves **Reporting as expected / Report overdue / Unknown** per VIN, no threshold in the frontend (M16 cadence-relative), last-report age colored by the server's verdict never by the number, per-attribute freshness independent of reporting state, "evidence at an instant, never a live session".
- The Sept 2 reviews' structural findings (filtered per §5.9): route registry as canonical source, Effective Access matrix computed-but-unrendered (build it — it sells the permission model), map-first direction, two-tier prose disclosure.

## 9. What carries over from our own work (team day-one assets)

| Asset | Role going forward |
|---|---|
| `Code/smartsense-platform` | **The production seed.** Serverpod 4 workspace: real AWS IoT mTLS ingest of fleet telemetry, typed telemetry store, Annex-1 catalog, fleet/RV/owner/geofence/subscription/admin APIs, audit, FCM, auth; Flutter Web portal with imported pilot DB (71 RVs / 14 days); Jaspr port underway (`JASPR-PORT.md`, spike results). The repo the team onboards into. |
| RevD spec (+ CHANGES) `Docs/` | The governing alerts/J3309 HMI spec (gateway-primary, dual BLE streams, Event-ID dedup, HMI state machine, timing targets). §17's engineering confirmations = the open firmware questions. |
| `Dimitar/` (devkit + `tpms-sim` + outbox letters) | Working no-cloud sensor→gateway→app chain = the GM-J3309-03 evidence path in embryo. The outbox letters double as onboarding docs for BLE/pairing gotchas and open AWS questions. Dimitar = Steiner's firmware/hardware engineer. |
| `Code/Drive-Pressure` | Direct-mode reference: encrypted Findy adverts read directly on iPhone, background scan, CarPlay, local push; only written-down AES-128-CCM packet layout (official `findy-tpms-ble.docx` is a revision behind). |
| `Code/Mobile-App` | The taken-over Flutter app (active, BLE demo mode). Base or donor for the overhaul decision. |
| `SERVERPOD-ARCHITECTURE.md` | Best single backend onboarding read, **with caveats**: portal is Jaspr-first now (doc says Flutter Web), alert engine moved forward from P4 (RevD + alerts spec exist), re-check Serverpod 4 release status. |
| `CHANGES-TO-STEINER-SYSTEMS.md` | Live compliance log; standing rule: no changes to Steiner AWS without green light. Teammates must follow it. |
| `Code/Gateway-Sim` (Toit rig) | Fallback simulator; superseded by the real devkit + `tpms-sim`. Keep for the upstream issue drafts. |
| The three PDFs (Proposal / Frontend Response / Evidence Pack) | Their arguments won; background only. Reuse the Evidence Pack's measurement methodology for future client-facing proofs. |
| `Code/Backend-App`, `Code/Admin-Portal` | Legacy clones, reference only. `Backend-App/docs/Gateway_proto.csv` remains the parameter-ID source until reseeded. |
| `GATE0-TALKING-POINTS.md` | Gate 0 is past; surviving items → backlog: #5 claim spoofability, #6 timestamp trust, #8 Google key, #9 truth states, #11 Domain Invariants doc (now in progress, §10). |

## 10. Domain-invariants seed (feeds the Domain Invariants & Vocabulary document)

> **Superseded by `DOMAIN-INVARIANTS-AND-VOCABULARY.md` (v0.1 draft, Sept 7)** — the list below was the raw harvest; the standalone document numbers, cites, and extends it, and adds the controlled vocabulary. Edit there, not here.

Harvested from the register + review archive + handoffs; each will get a number, controlled vocabulary, and register/doc citation in the real document:

1. One condition episode = one canonical AlertEvent; repeated telemetry, reminders, BLE-then-cloud sync, and late duplicate uploads never create a second episode. Backfill of a cleared episode never re-enters Active Conditions.
2. An episode is permanently bound to the rule/definition version that evaluated it; later edits never rewrite history; severity is stored per version; no client-side escalation from magnitude/duration/age.
3. Exactly one local evaluator per Product Mode (Gateway in gateway mode, the app in direct mode); the cloud detects communication absence only — "an offline source cannot report its own outage" — and never re-evaluates local thresholds. Clear is evaluator-driven; the portal has no resolve/ack action.
4. Severity determines nothing else (no channel, ack requirement, cadence, snooze, All-Clear, escalation).
5. Recipient = account, not class. Ack lives on the AlertEvent keyed by (event, recipient account); it stops that recipient's reminders only and never clears anything. Snooze is an interval; no reminder inside an active snooze; reminders stop at earliest of that recipient's ack or the episode's clear. Delivery attempts are explicit evidence records — "configuration is not evidence." Ack ⊥ delivery (each provable without the other).
6. Definition Disabled ≠ Gateway Clear; a configuration closure carries no recovery value (enforced down to a DB CHECK constraint).
7. Product Mode is server-resolved from the Model, never inferred from gateway presence. An absent capability is Not Available — never Offline/Stale/blank/zero. Stale ≠ Offline ≠ Not Available (≠ Unknown).
8. TPMS positions come only from the Model's declared position list (≤3 axles / 6 tires); position is a dimension beside the attribute (`tpms.pressure` + position), never part of the attribute id. One shared TPMS pressure/temperature definition across all positions; the triggering position is event data.
9. Freshness is server-resolved, per attribute, relative to configured cadence + grace (M16) — never computed client-side, never a universal threshold, never value-change age. Reporting state (Expected/Overdue/Unknown) is schedule-aware and separate from per-attribute freshness and from connectivity evidence.
10. Event time and cloud receive time are separate preserved facts (M20); filtering/sorting uses canonical event time; newer data is never overwritten by older (M13 delta rule: update only included attributes; absence ≠ null). Client timestamps are untrusted on the config plane (server-owned actors/timestamps) while source event time is preserved on the telemetry plane.
11. Sample interval ≠ reporting cadence; threshold events report immediately regardless of routine cadence (M12).
12. A verdict must be a function of its evidence, never drawn independently; Unknown never overrides determinate actionable evidence; missing data can never appear Healthy (M44); the system never converts ignorance into a claim.
13. Tenant switch invalidates all context-bound state; visibility is never the security boundary; search/sort/count follow the same authorization as content (row order and counts are inference channels); inaccessible objects are indistinguishable from nonexistent ones.
14. Claim state, owner activation (M8, independent fact), OEM VIN registration (M52, starts the included term), and gateway lifecycle are separate controlled dimensions; release keeps history; event-time recipient snapshots are never rewritten.
15. Configuration = desired/delivered/applied/reported with server-resolved convergence (M58); drift is visible, last-good preserved.
16. Immutable versioning applies to alert definitions, notification policies, and geofence geometry alike; historical events keep the version that produced them.
17. Local safety path (sensor → evaluator → HMI) never depends on cloud/LTE/push; ack/snooze never suppress the local warning; the safety Event ID is minted once and reused across BLE and LTE for dedup (RevD).

## 11. Questions for Tom (fold into next call)

> **Current asks live in `OPEN-QUESTIONS.md`** (CLIENT rows) — the register carries every reshape from the Sept 7–8 decision pass (viewer-local timezone proposal OQ-8, Sig-G severity bands OQ-5, customizable Fleet columns OQ-10, claim-stage proposal OQ-2, AirCheck-parity boundary OQ-4, Critical Alerts entitlement OQ-43). The list below stands where the register doesn't mark it reshaped.

0. **Missing source documents**: `05_J3309` §8 cites the J3309 Transfer Packet (Sept 3), Clause Matrix and Settings Review v0.1, Alerts Impact and Traceability Addendum v0.1, and `Annex_1_SmartSense_Gateway_Database_4_1.xlsx` — none delivered; our Annex-1 copy is version `_2`. Please send all four (they are authority #4/#5).
1. Confirm the legacy Django backend is retired for all users — we treat it as reference-only and consider the four PRE-B0 patches moot. *(Settled on our side, Sept 7; one confirming sentence wanted.)*
2. Confirm the claim/registration flows' place in the development order (they're absent from the new sequence; OD-009 direct-mode claim still open).
3. Opportunistic phone uplink when gateway LTE fails: our proposal, enabled by RevD's Event-ID design — does the product want it, and what does the gateway expose over BLE telemetry vs LTE?
4. Alert content, reshaped (OQ-1): production launches with an empty definition set and you author definitions in the admin panel — the only numbers we need from your side are the **per-Model TPMS safety values** (SSP per tire position, ΔP within the 20%-of-SSP rule, high-temperature limit), which J3309 evidence needs anyway. Same shape for the measurement catalogue (OQ-14): we seed it with the prototype's eight and you add datapoints in a governed Admin › Datapoints surface later — needs your nod for that new route.
5. VIN history end-state: current table for R1 with Candidate B ("three evidence lenses") earmarked — confirm.
6. Per-tire SSP on RV Models vs shared TPMS alert thresholds — confirm intended coexistence. Also: Fleet column set (open decision #12) and Significant-G default sort.
7. Foreground vs background claim boundary for the mobile local-warning path (J3309 evidence scoping) — agree in writing before evidence collection.
8. Apple Critical entitlement: has anyone approached Apple? If not, defer past R1.
9. Backend AWS Blueprint v0.8 / Mobile Blueprint v0.6: superseded or still live? (The new authority order doesn't rank them.)
10. Gentle, standing: gateway firmware source visibility (GM-J3309 evidence rows will eventually need firmware-side evidence from Dimitar's lane).

## 11a. Team working documents (added Sept 7)

GBot developers Bilal and Mert produced three documents on Sept 4, reviewed here Sept 7 — all three are sound and are now part of the team canon (`Team-Onboarding/6-Team-Working-Docs/`):

- **Route Register**: 47 routes with policy/protection/grid-query contracts and an S1–S6 build order (foundation → fleet spine → access admin → operational surfaces → sensor data → alerts; 34 routes in R1, 13 out). Correctly includes `distribution.deploy` (the gate the frontend spec dropped) and treats `device.command.execute` as "declared, unavailable — keep it that way." §5 additionally fixes the shared foundations: the seven portal failure states with exact copy (spine-style vocabulary; correlation id + contract version on every response — informs the OQ-12 envelope mapping), the six addressable child records that need no policy rows, the URL alias table to carry forward, and "unresolved active organization = invalid session, never a chooser." Two prototype details it extracts should not survive literally into production: the `cur:`-prefixed zero-padded-offset cursors (make cursors genuinely opaque) and the RV Owners default sort `attention` (same composite-ranking category the client deleted from Fleet as undecided — check it).
- **Domain Model**: 51 entities / 11 groups for Serverpod+PostgreSQL, 11 schema invariants matching ours, and four correctly-identified missing contracts (AlertEvent store, notification delivery storage, principal/session, Owner Activation). It resolves our OQ-12 PK half (public id `AD-0001` + per-org unique index) and designs OQ-7 correctly (SSP stored per position on the Model, explicitly fenced from alert thresholds). **Approved by Gazihan 2026-09-07** as the basis for schema work, with caveats on record: production cursors are opaque (OQ-37, resolved), the RV Owners `attention` default sort needs a decision before it's built (OQ-36), and Owner Activation stays unmodeled until OQ-6 is answered. The doc's own "Blocked on B1, B2 and B4" line: B1/B2 definitions are still owed by the team (OQ-34); B4 = the empty production datapoint catalogue — **reshaped 2026-09-07 (Gazihan): the catalogue is general and extensible**, seeded with the prototype's eight measurements, with later datapoints authored in a governed admin surface (OQ-14). Groups A/B/J can scaffold now, and group E can test against the seed; what remains client-side is blessing the extensible model and the new Admin › Datapoints route.
- **UX Review**: 26 walkthrough notes dispositioned; correctly refuses to let the Portal invent judgements (no client-side thresholds/colour-coding, no health-trend without a definition, Alerts stays read-only). Reveals the client already produced an **"R2 branch"** doing a two-tier disclosure pass — get that branch delivered before redesigning prose density. Its §7 changes sequencing, not just content: design tokens (border/hierarchy scale) and the single shared filter component move into S1, and the per-type event presentation must be settled **before S6** because it shapes the AlertEvent contract. Note 25 spots that the alert-authoring wizard resembles the Goodie Food wizard — reuse that reviewed pattern for the S6 authoring surface.

One process fix these surfaced: we now have **three parallel question registers** (our OQ-1..12, the team's B*/N* blockers, and the client's open PO decisions). Merge into one tracked list — see OQ-13.

## 12. Suggested team reading order

1. `ORIENTATION.md` → this document.
2. `00_Index` + `03_Serverpod_Handoff` (everyone), then per lane: backend → Alerts Backend Implementation Specification + contract spine + `SERVERPOD-ARCHITECTURE.md`; portal → `01_Frontend_Spec` + `Permission Model - Release 1.md` + accepted portal source + Sept 2 Screen-by-Screen review (with §5.9 filter); mobile → `04_Notifications_Mobile` + `05_J3309` + RevD spec + `Drive-Pressure` README + Dimitar outbox letters.
3. `Docs/SmartSense-Logic-Explorer.html` for register lookups as they come up (the frozen authority).
4. `Code/smartsense-platform` — clone, run, read `JASPR-PORT.md`.
5. `CHANGES-TO-STEINER-SYSTEMS.md` — the standing no-AWS-changes rule.
