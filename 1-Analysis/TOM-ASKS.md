# Asks for Tom — consolidated

GBot Software · September 8, 2026 · Gazihan's working list for the next call/message
One canonical list; tracking ids in brackets map to `OPEN-QUESTIONS.md`. Detail on any item: the register, or `OD-TRIAGE.md` §3 for the firmware ones.

## A. Send us

1. **The four J3309 documents your Sept 3 package cites but doesn't contain**: the J3309 Transfer Packet (Sept 3), Clause Matrix and Settings Review v0.1, Alerts Impact and Traceability Addendum v0.1, and `Annex_1_..._Database_4_1.xlsx` — our newest Annex-1 copy is v2. [OQ-28]
> Tom will check these documents and provide them

2. **The portal "R2 branch"** (the two-tier disclosure pass done after the handoff review) — we want to see it before touching page-density design. [OQ-27]
> Tom will see if he can get this to us.

3. **The real TPMS safety numbers per Model**: SSP for each tire position, the approved pressure-drop ΔP (within the 20%-of-SSP rule), and the high-temperature limit. These are what the Gateway will actually evaluate and what J3309 evidence gets measured against — nothing else in alerts is blocked on you anymore (see D below). [OQ-1]
> Tom will see if he can provide these.

## B. Confirm (one-sentence yeses)

4. The legacy Django backend is retired for all users — we treat it as reference-only and the old "PRE-B0" security patches as moot. [OQ-29]
> Confirmed with Tom, legacy backend is a reference for us.

5. Per-position SSP on RV Models coexists with the shared TPMS alert thresholds. (Two different numbers: SSP is the manufacturer's recommended cold pressure per tire position — vehicle spec, differs by axle; the alert threshold is the alarm trip point, and your September rule says there's one shared definition across all positions, never per-tire. We keep them strictly separate — nothing in alerts ever reads SSP as a threshold; they only meet in the governed J3309 derivation of the safety values from A3. Confirming, since the Aug 30 per-position SSP work and the Sept 3 shared-threshold rule never say in writing that both are intended.) [OQ-7]
> We will think about what's the best way to do this. No decision for now.

6. Overview's "Needs Attention" section is a triage list that hands off to Fleet with the reason carried along (that's how we're building it). [OQ-15]
> Tom will work on what exactly the definition of Needs Attention is

## C. Decide

7. **Direct-mode claim criterion** (your open item OD-009): what proves ownership for a phone-only vehicle — how many of the factory sensors must be heard over BLE, within what window, and what happens with a partial or replaced sensor set. Reminder of why it's sensitive: a proximity-only claim is spoofable and unsold inventory on a dealer lot is claimable. We're scheduling the claim/registration flows as their own build stage, so this is the one piece we need from you. [OQ-2 / OD-009]
> No decision here, yet. We'll think about this. Depends a lot on what we can expect from users/salesman/OEM practically. Security is probably not a big issue.

8. **Telemetry retention**: how long raw telemetry stays queryable, and whether an archive tier after that is acceptable. We'll bring a proposed tiering with AWS cost estimates — you approve rather than design. [OD-015]
> 3 years.

9. **Fleet default columns** (your open decision 12): we propose making columns user-customizable (current eight as default; Model Year, Status Age, Mileage, Location and a display-only Organization as options), so your decision shrinks to "what's in the default set." [OQ-10]
> Yes the ones visible in the prototype is the default set.

## D. Our proposals — react/bless

10. **Extensible measurement catalog**: we seed production with the prototype's eight measurements and you add new datapoints (new sensor kinds, units, bounds) later in a governed Admin › Datapoints surface — instead of anyone authoring a full catalog upfront. Needs your nod because it adds one admin route to the controlled route registry. [OQ-14]
> Yes

11. **No upfront alert-definition dataset**: production starts empty and you author alert definitions in the admin panel (the wizard you already reviewed). Nothing to deliver — just confirm you're comfortable. [OQ-1]
> Yes

12. **Timezones**: customer-facing timestamps display in the viewer's current local timezone (named zone shown), falling back to ET when unknown; UTC stays on tooltips. [OQ-8]
> Yes

13. **Significant-G severity**: admin-configurable magnitude bands (you define the bands like any governed config; each event is stamped from the bands in effect at event time and never re-graded later). Until then Sig-G severity stays "Not specified". [OQ-5]
> Yes

14. **Opportunistic phone uplink**: when a gateway's LTE is dead, the owner's phone can relay its data to the cloud over BLE — the firmware's single-Event-ID design already makes deduplication safe. Wants your product yes, plus one firmware fact (how complete the BLE telemetry stream is vs the LTE payload). [OQ-3]
> Yes

15. **Mobile background-warning boundary**: we match Sensata AirCheck at minimum — monitoring keeps working in background after first launch (survives lock, app-switching, between drives; ends only on phone restart or force-quit), with Critical Alerts delivered even in Silent/DND. We'll put this in a one-page claim boundary for sign-off before any J3309 evidence testing is paid for. [OQ-4]
> Yes

16. **Apple Critical Alerts entitlement**: needed for #15, granted per-app by Apple, and Sensata's grant is precedent for TPMS. It's requested through the Apple developer account the app ships under — likely yours, so let's start the application now. [OQ-43]
> GBot will provide instructions to Tom

## E. Firmware facts — via Dimitar, whenever convenient

No source code needed for any of these — just how the device behaves. Could you connect us or pass them along?

> Tom will ask Dimitar about these 
17. Routine-report mechanics: latest-value collapse, heartbeat, exact cadence behavior. [OD-013]
18. Rule delivery: how the gateway acknowledges/applies a delivered rule set, offline retry, and whether it reports the version it's running (for drift detection). [OD-020]
19. OTA release mechanics: image signing, staged rollout possibilities, install timing/retries, halting a bad rollout. [OD-034]
20. Rollback: previous-image storage, bootloader behavior on failed install, rollback constraints. [OD-035]
