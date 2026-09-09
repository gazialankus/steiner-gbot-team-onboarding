# SmartSense — Open Questions Register

GBot Software · canonical register, created September 7, 2026 · resolves OQ-13

> **The one list.** Every open question about SmartSense Release 1 lives here under one canonical `OQ-` number, whatever register it was born in — our analysis (OQ-1..13 kept their numbers), the team's blockers (`B*`/`N*`), the client's open Product Owner decisions, UX-walkthrough notes, or the August OD items. Aliases are recorded so any document citing an old id still resolves. New questions get the next number **here**; no other list may mint question ids. When a question closes, set Status with a date and one line of outcome — never delete the row.

**Decider key:** CLIENT = Tom / Product Owner · GBOT = Gazihan · TEAM = Bilal & Mert (GBot dev team) · JOINT = needs both sides.

## Blocking soon (answer before or during the next build stage)

| ID | Question | Decider | Aliases / source | Status |
|---|---|---|---|---|
| OQ-14 | DatapointDefinition catalogue. **Reshaped by GBot decision (Gazihan, 2026-09-07): the catalogue is general and extensible** — seed production with the prototype's eight review measurements (keeping their Annex-1 gateway-setting mappings) as the starter set; new datapoints (new sensor kinds, units, operators, bounds) are authored later through a governed admin surface (server-validated, versioned, audited), not committed upfront. Group E becomes testable against the seed. Remaining for CLIENT: bless the extensible model + the new Admin › Datapoints authoring surface (a route-registry addition, so a PO decision); note new datapoint *kinds* needing gateway-side evaluation still require firmware/ingest work — admin authoring alone covers the cloud plane | CLIENT | team **B4**; OD-016/M19 | **RESOLVED 2026-09-08** — Tom blessed the extensible catalogue + Admin › Datapoints route |
| OQ-1 | Alert Definition content. **Reshaped by GBot decision (Gazihan, 2026-09-07): no upfront dataset** — production launches with an empty definition set and the client authors definitions in Admin › Alert Definitions (the demo `RULES`/`NOTIF_DEFS` stay fixtures; timing fields stay blank-by-default per the governed docs). **Except the safety subset**: per-Model TPMS values — SSP per tire position, ΔP (≤20% of SSP) → low-pressure threshold, high-temperature limit — must be supplied and approved by the client before J3309 evidence collection (they are what the Gateway evaluates; GM-J3309-02 proves the exact effective version) | CLIENT | RR-002; 05 §6 controlled inputs; OQ-20 | TOM 2026-09-08: empty-start authoring **approved**; safety values — will see if he can provide |
| OQ-34 | Define team blockers **B1** and **B2** (named in the Domain Model status; definitions not yet shared) | TEAM | — | OPEN — placeholders |
| OQ-12 | Envelope convention: map `[ABIS]`'s fine-grained codes onto the seven portal failure states (Unauthenticated/Forbidden/NotFound/Validation/Conflict/Unavailable/Unexpected)? PK half already resolved (`publicId` + per-org unique index) | TEAM | spine `ContractEnvelope`; Route Register §5 | DESIGN PROPOSED |
| OQ-35 | Per-type event presentation set (tyre pressure vs Significant-G vs connection loss) — shapes the AlertEvent contract, settle **before S6** | TEAM | UX notes 22/23, §7 | OPEN |
| OQ-2 | Claim/registration flows. **Decided (Gazihan, 2026-09-07): we propose the plan** — claiming, gateway binding, replacement and recovery get their own scheduled backend stage (before any real-owner pilot); Tom is asked to decide the Direct-mode proof-of-ownership criterion (OD-009) with the spoofing/unsold-inventory risk restated | CLIENT | OD-009; GATE0 #5; HR-01 | STAGE PROCEEDS; OD-009 TOM 2026-09-08: thinking — depends on practical expectations from users/salespeople/OEM; he considers security a minor concern (our spoofing note stays on record) |
| OQ-6 | Owner Activation as an independent fact: backend definition + mobile contract + portal field (fixtures derive it from Claim — violates OWN-1) | JOINT | team **N7**; M8; L5 reconciliation | OPEN — do not model until answered |

## Product Owner decisions (client)

| ID | Question | Aliases / source | Status |
|---|---|---|---|
| OQ-10 | Fleet column set. **PROPOSAL to client (Gazihan, 2026-09-07): user-customizable columns** — current eight as default, the Merge-46 four (Model Year, Status Age, Mileage, Location) + display-only Organization available as options (reusing the Customize-columns pattern); PO decision 12 shrinks to "what is the default set" | client **PO decision 12** | **RESOLVED 2026-09-08** — customizable columns approved; default = the prototype-visible set |
| OQ-5 | Significant-G presentation. **Decided (Gazihan, 2026-09-07):** default sort = newest-first by event time. Severity: **PROPOSAL to client** — admin-configurable magnitude bands (governed, versioned config, same extensibility philosophy as OQ-14); the server stamps each event's severity from the band-config version effective at event time; historical events never re-grade when bands change (P-4). Until approved, severity stays "Not specified" | Sig-G Refinement; 1C §16 | **APPROVED 2026-09-08** — admin-configurable bands confirmed by Tom |
| OQ-7 | Confirm coexistence: per-position SSP on RV Models (recommended cold pressure) vs shared TPMS alert thresholds — design already separates them | team Domain Model; ledger §69 | TOM 2026-09-08: "no decision for now, we'll think about the best way" — coexistence unblessed; our fence (SSP never read as a threshold) stays as the safe default meanwhile |
| OQ-15 | Overview "Needs Attention" section. **Decided (Gazihan, 2026-09-07):** it is the triage entry point — reason chips + counts, click-through opens Fleet with the reason carried as a filter (per the PO's live-review direction); no duplicate grid ambition. Confirm with Tom in passing | PO review; UX note 6 | SHAPE STANDS; TOM 2026-09-08: he will work out the exact **definition** of Needs Attention — the current four-dimension union stays until his definition arrives |
| OQ-16 | Historical health / trend graphs. **Decided (Gazihan, 2026-09-07): parked post-R1** — deliberately-absent capability stays absent in R1; revisit with the client after launch with a real definition (time axis, population) | UX note 5 | PARKED POST-R1 |
| OQ-17 | Capabilities on vehicle history. **RESOLVED 2026-09-07 (Gazihan):** behind a control — keep the accepted build's header-popover pattern (capability applicability is a popover, never a column). Matches the client's accepted baseline; the team's "always visible" lean is overruled | UX note 17; L4-era VIN Detail | RESOLVED |
| OQ-19 | Subscription Plans dynamism. **POSITION RECORDED (Gazihan, 2026-09-07):** when built, plans are an admin-authored, versioned dynamic catalog — with the frozen snapshot rule intact: a VIN's terms are captured at OEM registration and later plan edits are never retroactive (M52/D-115). Module stays deferred | UX note 27; D-115 | POSITION RECORDED — module deferred |
| OQ-20 | SSP allowable range, precision, and step — **FOLDED into the extensible model (Gazihan, 2026-09-07):** datatype-only now, bound authored on the governed entry in admin later; no upfront client ask (SSP *values* per Model stay owed under OQ-1) | spine; DM | FOLDED |
| OQ-21 | `RvModel.modelYear` allowable range (invented 1990–2100 window removed) — **FOLDED into the extensible model (Gazihan, 2026-09-07):** datatype-only now, bound authored on the governed entry in admin later; no upfront client ask | spine; DM | FOLDED |
| OQ-22 | `RvModel.lengthFeet` range and precision (invented 60-ft ceiling removed) — **FOLDED into the extensible model (Gazihan, 2026-09-07):** datatype-only now, bound authored on the governed entry in admin later; no upfront client ask | spine; DM | FOLDED |
| OQ-23 | `RvModel.rvClass` enumeration — **FOLDED into the extensible model (Gazihan, 2026-09-07):** datatype-only now, bound authored on the governed entry in admin later; no upfront client ask; free text until an enumeration is authored | DM | FOLDED |
| OQ-24 | `Vin.includedTermMonths` permitted range (36 = controlled default) — **FOLDED into the extensible model (Gazihan, 2026-09-07):** datatype-only now, bound authored on the governed entry in admin later; no upfront client ask; store the snapshot, no invented validation | DM; M52/D-115 | FOLDED |
| OQ-25 | Model revision behaviour. **ENDORSED (Gazihan, 2026-09-07): protect the change, do not propagate** — a Model edit records the new value, never auto-pushes to fielded units; propagation is a future deliberate deployment feature (CONF-1, P-4) | DM | ENDORSED |
| OQ-26 | Audit on denials. **DECIDED (Gazihan, 2026-09-07): every authorization denial is audited** (OK/DENIED/CONFLICT/INVALID) — per SEC-7, the team, and ABIS. Mention to Tom for the record; not waiting on him | DM | DECIDED-OURS |
| OQ-36 | RV Owners default sort. **RESOLVED 2026-09-07 (Gazihan):** neutral default (VIN or owner name) until the client defines the ranking; `attention` offered as a sort option pending blessing — consistent with the client deleting Fleet's composite ranking as an unmade decision | Route Register §4 | RESOLVED — neutral default |
| OQ-8 | Timezone basis. **PROPOSAL to client (Gazihan, 2026-09-07): viewer-local time** — customer-facing timestamps render in the viewer's current local timezone (named zone shown), falling back to ET when the viewer's zone is unknown; UTC stays on tooltips; storage stays UTC. Ask Tom to confirm | Release T; Impl Note | **APPROVED 2026-09-08** — viewer-local display confirmed by Tom |
| OQ-9 | M43 quiet hours + always-on attribute. **Decided (Gazihan, 2026-09-07): backlog, reserve room** — not in R1; NotificationPolicyVersion reserves nullable columns now so later addition is config, not migration. M43 stays register-live | ledger §94 | BACKLOG — schema room reserved |
| OQ-11 | OD triage. **RESOLVED 2026-09-08 — `OD-TRIAGE.md`**: all 40 items sorted — 6 superseded, 17 ours, 11 post-R1, **6 need Tom** (OD-009 claim criterion, OD-015 retention terms, and four firmware-fact items OD-013/020/034/035 for Dimitar's lane); paste-ready shortlist in the triage doc §3 | Register ODs; review §6 | RESOLVED — see OD-TRIAGE.md |
| OQ-4 | Mobile warning-path claim boundary. **RESHAPED (Gazihan, 2026-09-08): match Sensata AirCheck BLE at minimum** — background monitoring works after first launch with permissions granted: survives lock, app-switching, and between drives; ends only on phone restart or force-quit (app must be reopened); Critical Alerts deliverable in background and through Silent/Focus/DND. The one-pager to compliance states this as the claimed behavior, with per-state *timing* claims left pending bench measurement (background BLE batching may stretch latencies). Requires the Apple Critical Alerts entitlement — see OQ-43 | GM-J3309-03; RevD; Sensata AirCheck precedent | **APPROVED IN PRINCIPLE 2026-09-08** — Tom yes on AirCheck parity; one-pager still to draft + sign before evidence spend |
| OQ-3 | **PROPOSAL** — opportunistic phone uplink in Gateway Mode when gateway LTE fails (enabled by RevD Event-ID dedup); needs yes + BLE-telemetry completeness answer | P-2; RevD | **APPROVED 2026-09-08** — product yes from Tom; remaining: BLE-telemetry completeness fact (Dimitar lane) + our ING/TIME design for relayed data |

| OQ-43 | **Apple Critical Alerts entitlement** — required to match Sensata AirCheck parity (OQ-4); Sensata's grant is precedent that Apple approves TPMS apps. Application goes through the Apple developer account the app ships under (Steiner's) — start the request now, not post-R1 | OQ-4; review §8 | INSTRUCTIONS DELIVERED 2026-09-09 (`APPLE-CRITICAL-ALERTS-REQUEST.md`), bundle ID confirmed `com.steinertech.rvapp` (existing app identity kept) — ready for Tom to submit |

## Deliveries & process (client)

| ID | Question | Aliases / source | Status |
|---|---|---|---|
| OQ-28 | Deliver the four cited-but-missing J3309 sources: Transfer Packet (Sept 3), Clause Matrix v0.1, Alerts Impact Addendum v0.1, Annex-1 v4.1 (we hold v2) | review §11 item 0 | TOM 2026-09-08: will check and provide |
| OQ-27 | Deliver the **R2 branch** (two-tier disclosure pass the team requested in the handoff review) before any prose-density redesign | UX note 2 | TOM 2026-09-08: will see if he can get it |
| OQ-29 | Confirm in one sentence: legacy Django backend retired for all users; PRE-B0 patches moot (settled on our side Sept 7) | review §4a | **RESOLVED 2026-09-08** — Tom confirmed: legacy backend is reference-only for us |

## Ours (GBot / team)

| ID | Question | Aliases / source | Status |
|---|---|---|---|
| OQ-18 | "Geometry page" in walkthrough note 24: **Gazihan does not recall raising it (2026-09-07)** — returned to the team: identify the walkthrough's author and what note 24 was actually looking at (geometry is currently a filter on the sites list, and Geofence Site Detail already shows geometry + version) | UX note 24 | RETURNED TO TEAM |
| OQ-37 | Production cursors: replace the prototype's `cur:`+zero-padded-offset with genuinely opaque cursors | Route Register §5 | **RESOLVED 2026-09-07** — approved by Gazihan: cursors are opaque, no decodable offset; implement at S1 |
| OQ-13 | One question register | — | **RESOLVED 2026-09-07 — this file** |

## Parked (from the August review archive, low urgency)

| ID | Question | Source |
|---|---|---|
| OQ-38 | Tenant/organization shell adjacency (client open #13) | Aug archive |
| OQ-39 | Export mechanics beyond `sensitive.export` (client open #14) | Aug archive |
| OQ-40 | Proxy/support acknowledgement (can support ack on a recipient's behalf?) | Aug archive |
| OQ-41 | Acknowledgement retention policy | Aug archive |
| OQ-42 | Cross-tenant query presentation for SmartSense Administrator | Aug archive |

---

*Cross-references: `DOMAIN-INVARIANTS-AND-VOCABULARY.md` Part D froze at OQ-13 and points here; `SEPTEMBER2026-DOCS-REVIEW.md` §11 is the client-facing ask list drawn from the CLIENT rows above. Team `B*`/`N*` ids and client PO-decision numbers remain valid as aliases but mint nothing new.*
