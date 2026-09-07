> **CAVEAT (added 2026-09-07):** This plan is dated 2026-08-09 and predates the September handoff. Three things changed since:
> 1. The portal is **Jaspr-first** (this doc says Flutter Web; the Jaspr decision came mid-August and is now confirmed in the client's `03_Serverpod_Handoff`).
> 2. The alert-rule engine moved forward from Phase P4 — the RevD spec and the client's Alerts Backend Implementation Specification now define it (see `1-Analysis/SEPTEMBER2026-DOCS-REVIEW.md`).
> 3. Re-check Serverpod 4's release status; this doc assumed public beta.
> Everything about the fixed device contracts, entities, ingest pipeline, and monorepo layout remains the working plan.

# SmartSense Backend on Serverpod — Architecture Plan

*v0.2, 2026-08-09 — supersedes v0.1. Strategy change: **greenfield build, no Django parity, no parallel-operation machinery** — SmartSense has never been opened to end users, so there is nothing to migrate users away from. We keep the AWS/IoT infrastructure and the deployed device fleet's contracts, and build the Serverpod solution directly. The old backend gets zero further investment and is switched off once the new system covers Steiner's internal needs.*
*Per the handover rules, items marked **[STEINER]** need their approval before build. GBot Software develops the mobile app (already agreed) and proposes to build the backend + portal (this plan).*

---

## 1. Strategy in one paragraph

Build the SmartSense cloud fresh on **Serverpod 4** (Gazihan's decision 2026-08-08; currently public beta — GBot runs stable Serverpod 3.x in production on Goodie Food). Reuse what is genuinely valuable and immovable: the AWS account and IoT plumbing (IoT Core, fleet provisioning, per-device X.509), the **deployed gateway fleet's contracts** (MQTT topics/payloads, OTA HTTP protocol — firmware in the field depends on them), the Firebase project, and the domain knowledge encoded in the parameter catalogs. Discard without ceremony: the Django application, its RE ST API shapes, its admin, the React portal prototype. The mobile app is ours and moves straight onto the generated Serverpod client as we develop it — **no compatibility layer, no golden-response tests, no parity gates, no shadow/parallel operation.** The only compatibility obligations are to hardware in the field, not to software users (there are none yet).

## 2. Fixed points (unchanged constraints)

| Contract | Detail | Why fixed |
|---|---|---|
| MQTT uplink | `devices/{serial}/tx`, JSON `{parameterID: value}`, param 257 = device UTC epoch | ~47 gateways in the field run this firmware (verified live 2026-08-07) |
| MQTT downlink | `devices/{serial}/rx`, `*SET,{id},{value}$` family; queued commands flushed when device connects, ≤2 s (spec §7.4) | same |
| Device identity & provisioning | Thing name = serial; claim-cert fleet provisioning template + Lambda hook; per-device policies | deployed fleet + Terraform |
| OTA HTTP | size probe + chunked download with `Content-Range` / `X-Total-Chunks` / `X-Chunk-Index`; params 51/55/52, states 53/54 | gateway firmware **[STEINER: add device auth]** |
| Annex-1 parameter IDs | numeric IDs are the law; catalogs seeded from `Gateway_proto.csv` | gateway spec (frozen) |
| Firebase project `smartsense-e011a` | sign-in (Google/Apple) + FCM | app + user accounts |
| AWS account/services | IoT Core, RDS, ECS, S3, SES, us-east-1 | Steiner's environment |
| Pending device-contract changes | topic namespace `ss/u|d`, Event-ID, 6→12 sensors, wire units | **[STEINER + vendor CRs]** — ingest edges are config-driven either way |

Explicitly **not** fixed anymore: the v1/v2 REST APIs, DRF response shapes, SimpleJWT tokens, Django admin behavior, the React portal. The app's API layer is replaced wholesale by the generated client (it was already isolated in one retrofit client + DTO folder).

## 3. Target topology

```
 Gateways (LTE, mTLS X.509) ──► AWS IoT Core ──MQTT──► ingest service (Dart, single-writer,
      unchanged topics                                  supervised ECS service w/ health checks)
      devices/{sn}/tx|rx                                   │  batched typed inserts, device timestamps
                                                           ▼
                                              PostgreSQL (RDS — new database)
                                              · telemetry_reading (monthly partitions, BRIN)
                                              · device_latest (shadow table for dashboards)
                                              · hourly/daily rollups · domain tables
                                                           ▲
 Flutter mobile app ──────► ALB ──► api service (Serverpod 4)
   (generated client)               · typed endpoints + WebSocket streaming
 Flutter Web portal ──────►         · firmware chunk routes (S3-backed, device-facing)
   (S3+CloudFront)                  · auth: email + Firebase IdP · FCM via HTTP v1
                                    · future-call jobs (rollups, OTA watchdog, dispatch)
```

Two deployables from one codebase: **`api`** (stateless) and **`ingest`** (single MQTT writer — preserves in-order command drain; supervised, alarmed; documented IoT-Rules→SQS fan-out path if scale ever demands). Both can run on Steiner's existing ECS next to the old containers until those are switched off; the old system gets no further work.

## 4. Monorepo layout (mirrors Goodie Food's proven shape)

```
smart_sense/
  server/steiner_server/       # Serverpod app: endpoints, ingest entrypoint, jobs, migrations
  packages/steiner_client/     # generated client — imported by portal + mobile app
  packages/shared_domain/      # pure Dart: Annex-1 parameter catalog, unit conversions,
                               #   alarm codes/severities, threshold logic, VIN checks
  apps/steiner_mobile/         # the existing Mobile-App, migrated in as it adopts the client
  apps/operator_web/           # Flutter Web portal (mockup = spec)
  tools/device_sim/            # gateway simulator: replays Gateway_proto.csv frames over MQTT
  tools/seed/                  # catalog seeding (Gateway_proto.csv) + optional import of
                               #   pilot registrations (orgs/users/71 RVs) from the old DB
```

`shared_domain` is the strategic asset: ingest, portal, and the app's BLE layer all read the same parameter table and conversions. (In Goodie Food the equivalent package is what keeps three clients honest.)

## 5. Data model

Fresh Serverpod-native schema in a **new database** on the existing RDS instance. No ETL obligation: catalogs are seeded from `Gateway_proto.csv`; pilot registrations (organizations, users, RVs/wheels, gateways, sensors — small: 71 RVs) get an optional one-shot import script so the fleet doesn't need re-registration **[STEINER: want history?]**. Old telemetry (text EAV) is archived to S3 and not migrated.

- **Domain tables** (~20): Organization (+ closure table for the tree), OrganizationRole, RVModel/RVModelWheel, RV/Wheel, Gateway, Sensor, SIMCard, Supplier, Subscription, Firmware, ParameterDefinition/SensorField, GatewayCommand, FirmwareUpdateRollOut/Device, Geofence (geometry via Serverpod 4 **geography types** — circles finally evaluated)/GeofenceEvent/GeofenceSettings, FcmToken. Implicit Django `save()` logic becomes explicit services (wheel materialization from model templates, role sync, rollout stamps).
- **Telemetry** (the design their review demands):
  `telemetry_reading(gateway_id, sensor_id, field_id, ts /*device time, param 257*/, value_num, value_text?)` — monthly native partitions, BRIN(ts), one batched INSERT per uplink frame; `device_latest` upserted per frame (what every dashboard reads); hourly/daily rollups; retention 90 d raw / 2 y hourly **[STEINER: confirm]**. Scale check: 10k gateways @ 15-min ≈ 11 msg/s — comfortable; 50k adds read replica, still no exotic store.
- **Alarm** carries forward-compatible columns now: `classification`, `rule_id/rule_version`, `event_id` (NULL until firmware CR), `acknowledged_at/by`, `snoozed_until`; ingest updates open alarms instead of inserting per report (kills the alarm-storm bug).

## 6. AuthN / AuthZ

- **AuthN:** Serverpod auth module; email+password and social via **Firebase IdP** (same project, same Google/Apple UX — and the pattern GBot already runs in Goodie Food with `serverpod_auth_idp`). FCM via **HTTP v1 API** with the existing service-account JSON; ID-token verification via Google JWKS. No JWT compatibility layer — the app adopts Serverpod sessions with the generated client. Preserve the `apple.id` private-relay email-blanking behavior.
- **AuthZ:** one model, enforced. `ScopeService` (org-tree closure + RV ownership) required by every fleet-touching repository method — the handover doc's invariant becomes a compile-visible pattern + CI grep. Roles are **data with capability codes** (mockup's roles × resources matrix), consulted by the API (unlike today), extensible into plan entitlements later.

## 7. MQTT ingest

Dart `mqtt_client` over mTLS to `iot.dev.halepu1969.com:8443` (new client cert — the committed one gets revoked **[STEINER: rotation]**). Pipeline per message: parse → device time (257) → idempotency (serial+257+hash; QoS-1 dup guard) → batched telemetry insert + `device_latest` upsert → alarm decode (positional list now; Event-ID strategy when CR lands) with open-alarm dedupe → gateway status/last_seen → **command drain ≤2 s** → `MessageCentral` events (`telemetry.{serial}`, `alarm.created`) feeding streams, push dispatch, and geofence evaluation. Old Django listener can keep running in parallel harmlessly during development (different client-id; its DB is disposable); we stop it whenever convenient.

## 8. Jobs (Serverpod future calls — no Celery, no Redis broker)

| Job | Trigger |
|---|---|
| Push dispatch | `alarm.created` event (immediate); FCM result actually checked before marking notified |
| Geofence evaluation | at ingest on GPS params (event-driven; circles + polygons via geo types) |
| Rollups + retention | nightly |
| OTA watchdog | per in-flight rollout — timeouts/retries (today nothing advances rollouts at all) |
| Subscription expiry | daily |

## 9. API surface

Native Serverpod endpoints only, module-per-domain; every portal list view gets a **pre-joined row DTO** with server-side filter/sort/page; **streaming methods** power live surfaces (`watchFleetOverview`, `watchAlerts`, `watchGateway`); exports generated server-side to S3 presigned URLs. Device-facing HTTP: firmware size/chunk routes (S3-backed), provisioning hook Lambda upgraded to actually check serials against the DB **[STEINER: enable enforcement]**. No REST compatibility layer of any kind.

## 10. Phases

| Phase | Content | Exit |
|---|---|---|
| **P0 — Foundations** (~1–2 wks) | Monorepo + CI (GitHub OIDC → ECR); spike proving the risky bits at once: models/codegen on Serverpod 4 beta, one streaming endpoint, Firebase ID-token verify, FCM v1 send, MQTT subscribe to dev IoT; device simulator; catalog seeding | Spike demo; beta fitness verdict |
| **P1 — Live platform core** (~3–5 wks) | Ingest + telemetry store live on the real fleet; auth; org/fleet/gateway/sensor domain + services; portal MVP: fleet board, gateway/RV detail, alerts — streaming live data | Portal shows the pilot fleet live; Steiner internal users can monitor |
| **P2 — App on Serverpod** (~2–4 wks, overlaps P1) | Mobile app swaps retrofit+DTOs for `steiner_client`; auth via Serverpod; push pipeline end-to-end verified (fires a real FCM on a real alarm) | App on new backend; old backend has no remaining consumers |
| **P3 — Back office + switch-off** (~4–6 wks) | Provisioning flows, firmware campaigns (OTA that actually completes), geofence editor (geo types + map), permission matrix, catalogs admin | Steiner confirms internal needs covered → **Django/Celery/React portal switched off**; old DB archived |
| **P4 — Roadmap features** | Alert-rule engine (scope precedence, versioned rules, ack/snooze/escalation), notification cadences, distribution rings, subscription entitlements — per client roadmap order | ongoing |

No parity checklists, no rollback theater: until P3's switch-off, the old system simply keeps running untouched (zero investment) as a fallback for internal staff. Commercial launch to end users happens on the new stack only — **gate: Serverpod 4 stable release (or explicit [STEINER] sign-off)**; development and the pilot fleet run on the beta where issues cost hours, not customers.

## 11. Risks

| Risk | Mitigation |
|---|---|
| Serverpod 4 beta instability | Pilot-fleet-only exposure until stable; P0 spike as fitness test; pin builds; GBot's Serverpod 3.x production experience (Goodie Food) as fallback know-how; framework-facing code isolated |
| Single-instance ingest | Supervised + health-checked + alarmed (unlike today's nohup); SQS fan-out path documented |
| Device-contract decisions pending | Ingest edges config-driven; nullable schema columns pre-added |
| Pilot telemetry continuity during transition | Old listener can run in parallel harmlessly until we choose to stop it; new store captures from P1 |
| Steiner staff workflow gap before P3 | Old admin stays available untouched until switch-off is agreed |

## 12. Decisions needed from Steiner

1. Approve backend + portal on Serverpod (this plan) — the app is already GBot's.
2. Deploy access to the DEV account (we hold ReadOnly).
3. Pilot data: import registrations into the new system, or re-register fleet? Telemetry history: archive-only OK?
4. Device-contract CRs with the vendor (topics, Event-ID, 6→12, units) — unchanged process, our side absorbs either outcome.
5. Rotation: IoT backend cert, committed secrets, TPMS AES keys (with Sensata).
6. Retention policy; commercial-launch gate acknowledgment (Serverpod 4 stable).

## 13. Immediate next actions

1. P0 spike (§10) — also the demo for the Steiner meeting when Gazihan is back.
2. Draft the portal information architecture from the mockup (it is the spec).
3. Request deploy access + agree the client-id/cert plan for the new ingest.
4. Seed tooling: `Gateway_proto.csv` → catalogs; optional pilot-registration import script.
