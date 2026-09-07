# SmartSense — OD-001..040 Triage

GBot Software · September 8, 2026 · resolves OQ-11
Source: the frozen Register's Open & Deferred items (readable form: `Docs/SmartSense-Logic-Explorer.html`). Buckets reflect the recorded card text plus the decisions settled Sept 3–8 (see `OPEN-QUESTIONS.md`). Only the NEEDS-TOM rows ever reach the client.

## 1. Bucket summary

| Bucket | Count | IDs |
|---|---|---|
| SUPERSEDED | 6 | OD-001, 008, 011, 016, 024, 025 |
| OURS | 17 | OD-004, 005, 006, 012, 014, 017, 018, 019, 021, 022, 023, 027, 028, 029, 033, 037, 039 |
| NEEDS-TOM | 6 | OD-009, 013, 015, 020, 034, 035 (four are firmware facts best routed through Dimitar's lane) |
| POST-R1 | 11 | OD-002, 003, 007, 010, 026, 030, 031, 032, 036, 038, 040 |

## 2. Full triage table

| ID | Topic | Recorded status / lane | Bucket | Rationale |
|---|---|---|---|---|
| OD-001 | Subscription start timing (OEM-paid term starts at OEM VIN registration; Owner Activation Date separate) | RESOLVED BY M52 / D-115 | SUPERSEDED | Already resolved in register; VIN snapshot non-retroactivity frozen (M52/D-115). |
| OD-002 | Dealer / service-center architecture (assignment, permissions, duration, workflows) | DEFERRED | POST-R1 | Intentionally deferred module (M7/M35/M66); no R1 dependency. |
| OD-003 | Maintenance effects at Owner Activation | DEFERRED | POST-R1 | Rides with the deferred maintenance module (OD-040/M70). |
| OD-004 | Mobile/Gateway BLE refresh and alert latency vs J3309 test definition | WIP — DEVELOPER CONFIRMATION | OURS | Engineering validation during build/test; gateway-side latency numbers come from Dimitar's firmware lane. |
| OD-005 | Direct TPMS routine cloud upload cadence, alert-event upload timing, buffering limits | WIP — DEVELOPER CONFIRMATION | OURS | App/server engineering design decision (cadence + buffering defaults, values as config). |
| OD-006 | Missing-TPMS-sensor trigger/clear timing, severity, hysteresis | WIP — TECHNICAL | OURS | Engineering defaults; value choices land in the admin-configurable alert-definition surface, not an upfront constant list. |
| OD-007 | New-owner visible historical fields on retail transfer | DEFERRED | POST-R1 | Register marks it deferred; when picked up, the exact field list is Tom's product/privacy call. |
| OD-008 | Direct TPMS background BLE behavior (iOS/Android, user-facing limitations) | WIP — DEVELOPER CONFIRMATION | SUPERSEDED | Mooted by the mobile-background decision: Sensata AirCheck parity is the floor (background after first launch, dies on restart/force-quit) + Critical Alerts application (OQ-4/OQ-43); residual OS validation is routine dev QA. |
| OD-009 | Direct TPMS activation sensor-presence requirement (count/timing/presence criterion) | OPEN — PRODUCT / DEVELOPER DETAIL | NEEDS-TOM | Explicitly being put to Tom per the claim-flow decision (OQ-2); architecture (local BLE proof of the VIN's factory sensor set) already approved. |
| OD-010 | Cloud ownership-transfer role matrix (dealer/service permissions) | DEFERRED WITH M7 | POST-R1 | Explicitly deferred with the dealer/service module. |
| OD-011 | Gateway cloud-fallback stale threshold | RESOLVED BY M16 | SUPERSEDED | Resolved in register; reporting-status pivot makes freshness cadence-relative with configurable grace — no hard-coded threshold. |
| OD-012 | Gateway App background BLE validation (OS-specific support/limits) | WIP — DEVELOPER CONFIRMATION | OURS | OS-behavior validation task; policy envelope set by the AirCheck-parity floor. |
| OD-013 | Gateway routine-report implementation details (latest-value collapse, heartbeat, cadence mechanics) | WIP — DEVELOPER CONFIRMATION | NEEDS-TOM | Requires gateway firmware facts (vendor-held); route gently via Tom to Dimitar's lane. |
| OD-014 | Ingestion / invalid-data handling (schema, auth, normalization, reject diagnostics) | WIP — DEVELOPER CONFIRMATION | OURS | Server-side design GBot owns in the Serverpod rebuild; only the gateway payload spec needs confirming with Dimitar. |
| OD-015 | Historical telemetry retention periods / archive tiers / audit storage policy | WIP — BUSINESS / DEVELOPER | NEEDS-TOM | Retention duration is a business/cost/compliance term only the client can set; GBot proposes tiers with cost estimates. |
| OD-016 | Canonical attribute catalog detailed population | WIP — SOURCE DATA | SUPERSEDED | Reshaped (OQ-14): general extensible catalog seeded with the prototype's eight measurements; later entries authored in a governed admin surface, bounds per-entry. |
| OD-017 | Timestamp synchronization / trust / fallback | WIP — DEVELOPER CONFIRMATION | OURS | Server-side trust/fallback design is engineering; gateway clock behavior fact-checked with Dimitar. |
| OD-018 | Mileage continuity on Gateway replacement | WIP — DEVELOPER CONFIRMATION | OURS | Solvable server-side (server-held accumulated mileage + offset on swap); firmware capability check with Dimitar only. |
| OD-019 | GPS accuracy / filtering / impossible-jump handling | WIP — DEVELOPER CONFIRMATION | OURS | Server ingestion-filter design; thresholds as configuration per the M16 config-not-constants pattern. |
| OD-020 | Rule distribution / configuration-drift mechanics (delivery ack, retry, versioning, drift detection) | WIP — DEVELOPER CONFIRMATION | NEEDS-TOM | Server-side versioning is ours; on-device rule application/ack is gateway firmware fact — Dimitar's lane, asked through Tom gently. |
| OD-021 | Alert clear / hysteresis / re-arm / anti-flapping | WIP — DEVELOPER CONFIRMATION | OURS | Engineering mechanics; values absorbed into admin-configurable alert definitions; gateway-side hysteresis facts via Dimitar. |
| OD-022 | Reminder behavior while an active alert loses communication | WIP — DEVELOPER CONFIRMATION | OURS | Notification-engine design decision, consistent with the R1 blank-by-default whole-minute timing model. |
| OD-023 | Alert severity labels / taxonomy | WIP — PRODUCT / DEVELOPER | OURS | Design-time proposal following the Sig-G pattern (admin-configurable, versioned at event time); lightweight client sign-off in review, not a blocking ask. |
| OD-024 | Communication-loss trigger / clear / severity / hysteresis | WIP — DEVELOPER CONFIRMATION | SUPERSEDED | Largely mooted by the reporting-status pivot: server-resolved Expected/Overdue/Unknown, cadence-relative with configurable grace (M16); values are configuration, remaining wiring is routine engineering. |
| OD-025 | Multi-user alerts / notifications / acknowledgement (secondary delivery, multi-phone, snooze, ack) | WIP — DEVELOPER CONFIRMATION | SUPERSEDED | Decided in the Sept 3 handoff: recipient-specific ack/snooze in whole minutes; R1 recipient classes fixed, mobile push only. |
| OD-026 | Notification channel expansion (SMS/email, consent, delivery evidence) | WIP — DEVELOPER CONFIRMATION | POST-R1 | R1 notification policy is mobile push only; SMS/email is a later channel expansion. |
| OD-027 | Duplicate-notification storm safeguard | WIP — DEVELOPER CONFIRMATION | OURS | Pure engineering safeguard (dedup without suppressing legitimate new AlertEvents); M43 quiet-hours schema room already reserved. |
| OD-028 | Per-unit health labels / precedence presentation | WIP — PRODUCT / DEVELOPER | OURS | Semantics defined; labels/precedence presentation is a portal UX proposal reviewed with the client, not a blocking product question. |
| OD-029 | Fleet-health population inclusion / exclusion (retired/deactivated VIN denominator) | WIP — PRODUCT / DEVELOPER | OURS | Lifecycle dimensions defined; GBot proposes a default (exclude retired/deactivated, visible filter) alongside the customizable-columns proposal (OQ-10). |
| OD-030 | Advanced analytics and AI product capability | DEFERRED / FUTURE PHASE | POST-R1 | AI is an explicitly deferred phase; current scope is display/history/export. |
| OD-031 | Subscription / entitlement architecture (billing, renewal/lapse/grace, plan transitions) | WIP — PRODUCT / DEVELOPER | POST-R1 | Subscriptions module deferred; plans-as-dynamic-catalog position recorded (OQ-19); included-term start already defined (M52). |
| OD-032 | Permanent Essentials safety-floor feature list beyond TPMS | WIP — PRODUCT / DEVELOPER | POST-R1 | Only matters when entitlement gating ships with the deferred subscriptions module; the eventual list is Tom's business call — queued with that module. |
| OD-033 | Flutter flavor vs server-runtime configuration boundary | WIP — DEVELOPER CONFIRMATION | OURS | Pure app/server architecture decision (GATEWAY vs DIRECT_TPMS product modes); no client input needed. |
| OD-034 | Gateway firmware OTA release mechanics and health gates (signing, rings, rollout %, halt criteria) | WIP — FIRMWARE / DEVELOPER | NEEDS-TOM | Firmware source is vendor-held/sensitive; needs firmware-side facts — Dimitar's lane, raised gently through Tom. |
| OD-035 | Gateway firmware rollback mechanics (image storage, bootloader, failed-install recovery) | WIP — FIRMWARE / DEVELOPER | NEEDS-TOM | Same as OD-034: bootloader/image facts only the firmware side holds. |
| OD-036 | Infrastructure failure / disaster recovery (redundancy, RTO/RPO, failover, runbooks) | WIP — DEVOPS / DEVELOPER | POST-R1 | Formal DR program is a deferred workstream; R1 carries baseline AWS backups within GBot's remit. |
| OD-037 | Geofence implementation / advanced logistics details | WIP — PRODUCT / DEVELOPER | OURS | Geometry/boundary/debounce/dwell are implementation design choices; the optional advanced transit/logistics tail is effectively post-R1. |
| OD-038 | OEM external API | DEFERRED / NEXT PHASE | POST-R1 | Explicitly not a Phase 1 priority; architecture merely stays future-ready. |
| OD-039 | Bulk export implementation mechanics (formats, file limits, async delivery) | WIP — DEVELOPER CONFIRMATION | OURS | Card marks it a Phase 1 requirement; formats and delivery mechanics are engineering design decisions. |
| OD-040 | Maintenance scheduling detailed workflow | DEFERRED / FUTURE PHASE | POST-R1 | Maintenance module explicitly deferred until prioritized. |

## 3. NEEDS-TOM shortlist (paste-ready)

**OD-009 — Direct TPMS activation: sensor-presence criterion.** The approved design requires local BLE proof of the VIN's factory sensor set before a direct-mode claim completes. We need your call on the exact criterion: how many of the factory sensors must be seen, within what time window, and what happens when a unit ships with a partial or replaced sensor set. This gates the direct-mode claim flow we are scheduling now.

**OD-013 — Gateway routine-report behavior.** To build ingestion correctly we need the gateway's actual reporting behavior confirmed from the firmware side: whether repeated readings collapse to latest-value, how the heartbeat/check-in works, and the exact cadence mechanics. This is Dimitar's territory — could you connect us or pass the questions along?

**OD-015 — Telemetry retention periods and tiers.** How long must raw telemetry stay queryable, and is an archive tier acceptable after that (with what access expectations)? This is a business/cost decision; we will bring a proposed tiering with AWS cost estimates for you to approve rather than asking you to design it.

**OD-020 — Rule delivery to the Gateway.** Server-side rule versioning is on us, but we need firmware facts on the device side: how the gateway acknowledges and applies a delivered rule set, retry behavior when offline, and whether it can report the version/hash it is running (for drift detection). Another one for Dimitar's lane when convenient.

**OD-034 — Gateway firmware OTA release mechanics.** Since the firmware source sits with the vendor, we need the facts of how OTA works today: image signing, whether staged rollout (rings/percentages) is possible, install timing, retries, and how a bad rollout is halted. We only need enough to build the cloud-side release controls around it.

**OD-035 — Gateway firmware rollback.** Companion to the OTA question: does the gateway keep a previous image, what does the bootloader do on a failed install, and what compatibility constraints apply to rolling back? This determines how much rollback safety we can offer from the cloud side versus what must live in firmware.
