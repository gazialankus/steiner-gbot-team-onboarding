# SmartSense — Domain Invariants & Vocabulary

**v0.1 DRAFT** · GBot Software · September 7, 2026

> **What this document is.** A derived working artifact under the frozen baseline: it **restates** the controlled decisions in one place with one name per concept and numbered, testable invariants. **It does not decide.** Where it goes beyond the baseline, the item is explicitly marked `PROPOSAL`. On any suspected disagreement with the sources below, the sources win and this document gets fixed.
>
> **How to use it.** Use these terms — and only these terms — in code identifiers, schemas, endpoint names, UI copy, commits, and discussion. The "banned" column lists words we've caught drifting; if you reach for one, use the controlled term instead. Cite invariants by ID in reviews ("this violates NOTIF-4"). Every invariant should eventually map to at least one automated test.

**Source key:** `[M#/D#/OD#]` = Critical Logic Register v0.10 + Process Spec v0.8 (frozen authority #1) · `[00..05]` = September 3 handoff docs · `[ABIS]` = Alerts Backend Implementation Specification r5 · `[PM]` = Permission Model – Release 1 · `[RevD]` = TPMS RealTime Alert & J3309 HMI Developer Spec Rev D · `[AA–AF]` = August release notes (accepted semantics) · `[GBot]` = our decision, confirmed by Gazihan.

---

## Part A — Design principles

**P-1 · The system never converts ignorance into a claim.**
Unknown is an honest first-class answer. Missing data is never rendered as Healthy, zero, or a guess; Unknown never overrides determinate evidence; a verdict must be a function of its evidence. `[M44, AE, AF]`

**P-2 · Opportunistic resilience — "roll with the punches."** `[GBot]`
The system pursues its goals through the best currently available path and never treats a component failure as an excuse not to deliver. Backend down → the local safety path still warns the user. Gateway LTE dead → the phone can carry data up (`PROPOSAL`, see OQ-3). Push undeliverable → the BLE path is unaffected. Guardrail: opportunism degrades latency, coverage, or precision — **never honesty** (P-1) and never evidence integrity (EVT-6). A fallback path reports itself as what it is.

**P-3 · The server is the authority; clients render.**
Authorization, validation, timestamps, actors, freshness verdicts, counts, ordering, population resolution, and health roll-ups are computed server-side. A hidden button is not authorization; local storage is not a security boundary. `[01 §1, 03 §3, ABIS]`

**P-4 · History is immutable; change creates versions.**
Published definitions, policies, and geofence geometry are never mutated — a change is a new version. Events keep the version that produced them. The past is never rewritten by the present. `[D-107, M23, ABIS, 02]`

**P-5 · The local safety path stands alone.**
TPMS Sensor → local evaluator → HMI warning must work with cloud, LTE, push, and portal all dead. Remote layers add awareness and workflow; they are never prerequisites for the warning. `[04 §1, 05 §1, RevD]`

---

## Part B — Controlled vocabulary

### Fleet & identity

| Term | Meaning | Banned / near-misses |
|---|---|---|
| **VIN** | The unit's identity; one towable RV | "vehicle record", "unit id" |
| **RV Model** (Model) | Catalog entity that fixes Product Mode and Tire Layout; every VIN inherits both | "vehicle type", "template" |
| **Tire Position** | Model-declared position id (ODSF, DSF, …) + axle; ≤ 3 axles / 6 tires; never derived from a count | "tire number", "wheel index" |
| **Gateway** | The ESP32 LTE/GPS/BLE device; the local evaluator in Gateway Mode | "hub", "device" unqualified |
| **TPMS Sensor** | The BLE tire sensor | "tag", "beacon" |
| **Organization** | Tenant-scoped org; types SMARTSENSE / OEM / DEALER / SERVICE_CENTER / FLEET | "company", "account" |

### Modes & capabilities

| Term | Meaning | Banned |
|---|---|---|
| **Product Mode** | `GATEWAY` \| `DIRECT_TPMS`; fixed by the Model; server-resolved, never inferred from gateway presence | "app mode", "has gateway" |
| **Capability** | A data/feature family applicable under a mode `[D-121]` | — |
| **Not Available** | Capability not applicable in this mode; a first-class state | rendering as Offline/Stale/blank/zero |

### Telemetry & time

| Term | Meaning | Banned |
|---|---|---|
| **Reading** | One governed measurement value observed at a source | "data point" for a value |
| **DatapointDefinition** | Governed catalog entry (id, unit, classification, applicability); server-discovered | client-invented measurement names |
| **Event Time** | When it happened at the source; the canonical time axis | unqualified "timestamp" |
| **Receive Time** | When the cloud accepted it; separate preserved fact | conflating with Event Time |
| **Sample Interval** | Local measurement rate (e.g. 15 min); a device-local fact | calling it a reporting rate |
| **Reporting Cadence** | Expected uplink schedule (e.g. ~4/day) | "polling rate" |
| **Delta Report** | Uplink carrying only changed parameters; absence of a key ≠ null `[M13]` | "partial update" treated as full |
| **Backfill** | Late arrival of older data `[M17]` | "sync" unqualified |

### Freshness & reporting state

| Term | Meaning | Banned |
|---|---|---|
| **Freshness** | Per-attribute observation age vs configured cadence + grace; server-resolved `[M16]` | value-change age |
| **Stale** | Attribute out of its freshness bound; a **symptom** | using it for device connectivity |
| **Offline** | Connectivity evidence about a *device* (confirmed offline / reachable / no evidence); a **cause** | using it for VIN reporting state |
| **Reporting Status** | Per-VIN: `Reporting as expected` \| `Report overdue` \| `Unknown` `[AD]` | **Online/Offline for VINs (dead)**, "connected" |
| **Unknown** | Evidence insufficient; visually distinct from None | defaulting to Healthy or zero |
| **Health** | Server roll-up: `Healthy` \| `Needs Attention` \| `Unknown` `[M44]` | computing it client-side |

### Alerts

| Term | Meaning | Banned |
|---|---|---|
| **Alert Definition** | Versioned governed config: measurement, operator, threshold, severity, persistence, Notification Policy | **"alert rule"** (legacy, deleted surface), "alarm config" |
| **Notification Policy** | Versioned config *inside* a Definition | standalone "notification rule" |
| **Threshold Violation** | An instantaneous comparison result | calling it an alert |
| **Condition** | A persistent abnormal state per the persistence rules | "issue", "problem" in specs |
| **AlertEvent** | The one canonical record of one condition episode; bound to the evaluating Definition version | **"alarm"** (legacy Django term), bare "alert" for the record |
| **Significant-G Event** | Momentary historical event; never Active, no All-Clear | treating as a Condition |
| **Severity** | `Advisory` \| `Warning` \| `Urgent`; a label that determines nothing else | severity-driven behavior |
| **Gateway Clear** | Evaluator-reported return to normal; carries the recovery value | "auto-resolve" |
| **Definition Disabled** | Administrative configuration closure; **no recovery value** | "resolved", merging with Gateway Clear |
| **All-Clear** | The optional *message* sent when a persistent Condition clears | confusing with Gateway Clear (state vs message) |
| **Event ID** | Single identity minted at the evaluator, reused across BLE and LTE for dedup `[RevD]` | per-path ids |

### Notifications & recipients

| Term | Meaning | Banned |
|---|---|---|
| **Recipient** | An **account** (Primary Owner or an Authorized User) resolved for the VIN at event time `[AB]` | recipient *class* as the storage model, **"service queue"** (deleted `[X]`) |
| **Delivery Attempt** | A recorded actual attempt with outcome and timestamp | counting configured reminders as attempts |
| **Acknowledgement** | Per (AlertEvent, Recipient); stops that recipient's reminders only | "ack clears the alert" |
| **Reminder** | Policy-driven repeat; whole-minute cadence | — |
| **Snooze** | Per-recipient pause interval (start, whole-minute duration, expiry) | global snooze |
| **Local Safety Warning** | The BLE-path HMI warning; independent of everything above | "notification" for it |

### Ownership & subscription

| Term | Meaning | Banned |
|---|---|---|
| **OEM VIN Registration** | Plant registration; starts the Included Term `[M52]` | calling it activation |
| **Claim** (Retail Ownership) | The owner-claims-VIN state machine | "registration" for it |
| **Owner Activation** | An independent fact `[M8]` — *not* derived from Claim (see OQ-6) | deriving from claim state |
| **Authorized User** | Additional account on the VIN's roster | "sub-user" |
| **Included Term** | The 36-month VIN-snapshot subscription window | — |
| **Recipient Snapshot** | Event-time roster capture; never rewritten on ownership change `[L5]` | reading it as current truth |

### Configuration

| Term | Meaning | Banned |
|---|---|---|
| **Desired / Delivered / Applied / Reported** | The four configuration states `[M58]` | "synced" as a single bit |
| **Convergence** | `Current` \| `Pending` \| `Drift` \| `Failed` \| `Unknown`; server-resolved | — |
| **Drift** | Device reports other than desired | "out of sync" |
| **Effective Version** | The Definition/Policy version the evaluator is actually running | assuming latest = effective |

### Tenancy & access

| Term | Meaning | Banned |
|---|---|---|
| **Access Profile** | One of five defaults, or a custom narrowing with an immutable base-profile ceiling `[PM]` | "role" in UI/specs (route id `admin.roles` is legacy) |
| **Page Access** | `R` \| `RW` \| `N` \| `NA` per route per profile | role-name checks in code |
| **Capability Gate** | Named protected control (12 incl. `distribution.deploy`) | inventing gates ad hoc |
| **Withheld** | A value exists but this viewer may not see it; ≠ None recorded | rendering Withheld as empty |
| **Active Organization Context** | The single org bounding every query; switch invalidates all context-bound state | multi-org queries |

---

## Part C — Invariants

### MODE — Product Mode & capabilities
- **MODE-1** Product Mode is server-resolved from the Model, never inferred from Gateway presence or telemetry shape. `[M72, D-121]`
- **MODE-2** An absent capability renders as Not Available — never Offline, Stale, Failed, blank, or zero. `[D-121]`
- **MODE-3** Health never penalizes a Direct-mode VIN for capabilities it does not have. `[M46]`
- **MODE-4** Tire Positions come only from the Model's declared list; position is a dimension beside the attribute (`tpms.pressure` + position), never part of the attribute id. `[1E, Y §A]`
- **MODE-5** One shared TPMS pressure Definition and one shared TPMS temperature Definition apply across all Model-defined positions; a position selector is rejected server-side; the triggering position is event data. `[02, 03, ABIS §5]`

### TIME — time & trust
- **TIME-1** Event Time and Receive Time are separate, both preserved, never merged. `[M20]`
- **TIME-2** Filtering, sorting, and range queries over events use canonical Event Time. `[M20, L2]`
- **TIME-3** Newer data is never overwritten by older data, on any path (live, Backfill, phone-relayed). `[M10, M13]`
- **TIME-4** On the configuration plane, timestamps and actors are server-assigned; client-supplied ones are discarded **and the attempt is audited**. `[ABIS §4]` (Telemetry-plane source times are preserved per TIME-1 — these rules govern different planes and do not conflict.)
- **TIME-5** Storage is UTC; customer-facing presentation is ET with a named zone and UTC on tooltips (current approved presentation; broader timezone basis is OQ-8). `[T, Implementation Note]`

### ING — telemetry ingest
- **ING-1** A Delta Report updates only the attributes it contains; absence of a key means "no news", never null. `[M13]`
- **ING-2** Invalid data never overwrites the last valid value. `[M13]`
- **ING-3** Backfill of an already-cleared episode never re-enters Active Conditions and never creates a new episode. `[Phase 2 §25]`
- **ING-4** Sample Interval ≠ Reporting Cadence; a Threshold Violation reports immediately regardless of routine cadence. `[M12, AD §G]`

### FRESH — freshness & reporting state
- **FRESH-1** Freshness is server-resolved, per attribute, relative to configured cadence + grace. No universal threshold, no client-side computation. `[M16, 1E]`
- **FRESH-2** Freshness tracks observation age, never value-change age: an unchanged-but-reported value is fresh. `[M16]`
- **FRESH-3** Reporting Status is a per-VIN three-state verdict (`Expected`/`Overdue`/`Unknown`), independent of per-attribute Freshness and of device connectivity evidence — three separate columns, never merged. `[AD, AE, §93]`
- **FRESH-4** An age ("Last report 6h ago") is a fact; only the server's verdict colors it — never the number itself. `[AD §H2]`
- **FRESH-5** Settings have no freshness; commands have no value, freshness, or history. `[1F]`
- **FRESH-6** Detail views state evidence-at-an-instant ("LTE state at last report"), never a live session. `[AE §D]`

### EVT — events & episodes
- **EVT-1** One Condition episode = one canonical AlertEvent. Repeated telemetry, reminders, BLE-then-cloud sync, and late duplicate uploads never create a second episode. `[Phase 2 §25, RevD]`
- **EVT-2** An AlertEvent is permanently bound to the Definition/Policy versions that evaluated it; later edits never rewrite it; severity is stored per version. `[D-107, Phase 2 §4]`
- **EVT-3** Severity determines nothing else — no channel, ack requirement, cadence, snooze, All-Clear, or escalation. `[2A §1]`
- **EVT-4** A Significant-G Event is momentary and immutable: never Active, never cleared, no All-Clear, no severity invented from magnitude (severity is `Not specified` until the client decides — OQ-5). `[02, Sig-G Refinement]`
- **EVT-5** Definition Disabled is a configuration closure, distinct from Gateway Clear, carrying no recovery value — enforced down to the database (`CHECK` constraints). `[ABIS, 02 §5]`
- **EVT-6** Events come from evaluators; nothing is ever manufactured from configuration ("configuration is not evidence"). The Portal never fabricates event rows. `[2C, 03 §5]`
- **EVT-7** Event evidence (location fix, values at trigger) is never overwritten by present state; "no valid fix" is never substituted with a current position. `[AE §E, Sig-G Refinement]`

### SAFE — evaluation & the safety path
- **SAFE-1** Exactly one local evaluator per Product Mode: the Gateway in Gateway Mode, the app in Direct Mode. In Gateway Mode the phone is HMI only — its cross-check is optional diagnostics, never an alert source. `[Phase 2 §12, RevD C→D]`
- **SAFE-2** The cloud detects communication absence only ("an offline source cannot report its own outage"); it never re-evaluates local thresholds and never competes with the local evaluator. `[Phase 2 §12, 00 §1]`
- **SAFE-3** The Local Safety Warning works with cloud, LTE, push, and portal all unavailable (P-5); Acknowledgement and Snooze never suppress it. `[04, 05, GM-J3309-03]`
- **SAFE-4** The Portal has no resolve/acknowledge action on events — not even a disabled one; it is evidence, read-only. `[02 §5]`
- **SAFE-5** HMI state honesty: never show monitoring-active while disconnected, stale, or running an invalid profile; confirmed BLE connection loss surfaces within 1 s and persists until reconnection. `[RevD]`
- **SAFE-6** The 250/250/500 ms timing targets are internal engineering targets unless the compliance owner maps them to an approved requirement; claim language stays "J3309 conformity evidence in progress." `[05 §3, RevD]`

### NOTIF — notifications
- **NOTIF-1** Recipients are accounts, resolved per VIN at event time from the fixed Release-1 classes (Primary Owner + eligible Authorized Users); no individual-person selector; ids globally unique; two Authorized Users are always two recipients. `[04, AB, AC]`
- **NOTIF-2** Acknowledgement is per (AlertEvent, Recipient): it stops that recipient's reminders only, never clears anything, and is valid evidence with or without a successful delivery ("failure is not evidence against acknowledgement; acknowledgement is not evidence of delivery"). `[AA]`
- **NOTIF-3** Reminders stop at the earliest of that recipient's Acknowledgement or the episode's clear; no reminder fires inside an active Snooze. `[AB]`
- **NOTIF-4** Delivery Attempts are recorded facts with outcomes and timestamps; configured or pending reminders are never presented as attempts; a failed attempt is visible as failed. `[04 §3, 2C]`
- **NOTIF-5** Timing fields are positive whole minutes, blank by default — no 30/60/120 presets; Snooze values entered one at a time. `[02 §4, 04]`
- **NOTIF-6** All-Clear exists only for persistent Conditions, per Definition, bound to the original AlertEvent; a clear without All-Clear configured sends nothing. `[04, Z]`
- **NOTIF-7** Cross-path dedup keys on the Event ID minted at the evaluator: BLE-presented and push-delivered instances of one event are one event to the user. `[RevD, GATE0 item 1]`
- **NOTIF-8** When recipient identity is Withheld, presentation uses a non-correlatable per-event ordinal ("Authorized User 1"), never a stable cross-event id. `[AC §B]`

### VER — versioning & the alerts backend contract
- **VER-1** Definition and Policy versions are immutable after publication; version tables are append-only (no UPDATE/DELETE by grant and by policy). `[ABIS §1]`
- **VER-2** The only disable path is the atomic `disableAndClose` transaction (definition state + event closures + audit + version transition commit or roll back together); `saveVersion` with enabled→false is rejected; there is no public closure-recording endpoint. `[ABIS §2, §7; 03]`
- **VER-3** Optimistic concurrency uses row locks with expected-version comparison; conflicting writes fail safely, never lost-update. `[ABIS §6]`
- **VER-4** Production alert creation uses server-discovered DatapointDefinitions only; the client can never introduce a measurement name or unit. `[02 §6, 03 §4]` (Authoring the catalogue itself is a separate, governed admin workflow — the catalogue is general and extensible by decision `[GBot, 2026-09-07]`: seeded with the starter set, extended through a privileged admin surface with the same validation/versioning/audit discipline. That workflow does not weaken this invariant: an *alert* author still picks only from what the catalogue serves.)

### CONF — device configuration
- **CONF-1** Configuration state is Desired/Delivered/Applied/Reported with server-resolved Convergence; Drift is visible, never silently reconciled; last-good is preserved. `[M58]`
- **CONF-2** The evaluator runs the Effective Version, and evidence records which version that was. `[GM-J3309-02]`

### OWN — ownership & subscription
- **OWN-1** Claim state, Owner Activation, OEM VIN Registration, and Gateway lifecycle are four separate controlled dimensions; no field collapses them. `[L5, M8, M52]`
- **OWN-2** The Included Term starts at OEM VIN Registration, not at claim or activation; plan terms are VIN snapshots — later Model/plan edits are never retroactive. `[M52, D-115]`
- **OWN-3** Release of a VIN makes it claimable again and retains full history. `[M5]`
- **OWN-4** Recipient Snapshots are event-time captures, never rewritten on ownership change and never read as current truth. `[L5]`

### SEC — security & tenancy
- **SEC-1** No endpoint accepts a tenant or organization argument; both resolve server-side from the session. `[ABIS §2]`
- **SEC-2** One Active Organization Context bounds every query; an authorized-org *list* is never a query scope; a context switch invalidates all context-bound data, cursors, caches, and editors. `[01 §4]`
- **SEC-3** Out-of-scope and nonexistent objects are indistinguishable (`NOT_FOUND` for both); counts, row order, search, and sort obey the same authorization as content — they are inference channels. `[ABIS §3, T]`
- **SEC-4** A deep link authorizes nothing; every route re-asks page policy and object scope on load; page access is policy-based, never a hard-coded role-name check. `[01 §6, 02]`
- **SEC-5** Sensitive fields are separately gated and Withheld, never reconstructed from other values, never reaching exports the viewer isn't authorized for. `[01 §5, PM]`
- **SEC-6** Custom Access Profiles only narrow their immutable base profile — the ceiling is enforced server-side; UI omission is presentation, never enforcement. `[PM]`
- **SEC-7** Denials and conflicts are audited too. `[ABIS §1]`

---

## Part D — Open questions (tracked; do not resolve silently)

> **All open questions live in `OPEN-QUESTIONS.md`** (project root; `1-Analysis/` in the onboarding bundle) — one canonical `OQ-` number per question, with current status and reshapes. This document deliberately does **not** duplicate the rows: a copied list goes stale, and a stale copy presenting itself as current is exactly the failure mode this project polices. When an invariant here cites an `OQ-` id, resolve it against the register.

> **Companion:** the team's **SmartSense Domain Model** (Sept 4) carries 11 schema-level invariants consistent with Part C and is the schema companion to this document. When the two disagree, flag it — do not silently pick one.

---

*Change control: edits to Parts A–C require a source citation or a `PROPOSAL` tag plus an OQ entry. Gazihan owns reconciliation with the client; the frozen Register v0.10 + Spec v0.8 remain the product authority this document restates.*
