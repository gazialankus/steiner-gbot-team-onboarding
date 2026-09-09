# Requesting the Apple Critical Alerts Entitlement

Gazihan Alankuş — GBot Software · September 9, 2026 · for Tom

## What this is and why we need it

iOS "Critical Alerts" are a special notification class that plays sound and vibrates **even when the phone is in Silent, Focus, or Do Not Disturb** — exactly what a tire-pressure warning needs while someone is towing. Apple only grants this ability per-app, on request, for health/safety/security use cases. A TPMS app is squarely what the feature exists for, and there's direct precedent: Sensata's AirCheck BLE app holds this entitlement and delivers TPMS Critical Alerts from the background — the behavior we're matching.

The request must come from the **Apple Developer account the app will ship under** — that's yours, which is why this is in your hands. Everything technical after approval is ours; this request is the only step that needs you.

## The app to name in the request

Bundle identifier: **`com.steinertech.rvapp`** — the existing SmartSense app; the rebuilt app keeps this identity, so the entitlement carries over. (If we ever ship a second app variant, that one would need its own request — nothing to worry about now.)

## Steps

1. Sign in at **developer.apple.com** with an account that has **Account Holder** access to your team (Account Holder is the safe choice; Apple sometimes rejects requests from lesser roles).
2. Open the Critical Alerts request form: **https://developer.apple.com/contact/request/notifications-critical-alerts-entitlement/**
3. Fill in the app name and the bundle identifier `com.steinertech.rvapp`.
4. For the use-case description, you can use this (edit freely):

> SmartSense is a tire-pressure monitoring system (TPMS) for towable RVs. Bluetooth tire sensors and an in-vehicle gateway monitor tire pressure and temperature while the vehicle is being towed, and the driver's iPhone is the warning display. A rapid pressure loss or blowout condition is a time-critical road-safety event: the driver must be alerted within seconds even if the phone is in Silent or Do Not Disturb, which is common while driving. A standard or time-sensitive notification can be missed in exactly the situation the product exists for. Critical Alerts are used only for active tire-safety warnings (pressure loss, critical temperature, monitoring failure while towing), never for marketing or routine status. The system is designed toward SAE J3309 TPMS requirements, which mandate immediate driver warning. Comparable TPMS applications (e.g., Sensata AirCheck BLE) hold this entitlement for the same use case.

5. Submit. Apple reviews these by hand; reports suggest days to a few weeks, and they sometimes reply with follow-up questions — if they do, forward them to me and we'll draft the answer together.

## After approval

Nothing more on your side. We add the entitlement to the app build, request the user's permission properly in-app, and keep Critical Alerts strictly limited to active safety warnings (that restraint is also what keeps the entitlement safe at App Review time).
