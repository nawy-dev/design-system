# ADR 0005: Android via Capacitor with local notifications

- **Status:** Proposed (accepted for Taqweem; needs owner approval as a shared default for other Nawy apps)
- **Date:** 2026-11-19

## Context

Taqweem is a local-first, Arabic-first PWA with no backend. Web push cannot give reliable calendar reminders without a server, and iOS web reminders are limited. The owner wants real reminders and distribution on Google Play (Android first).

## Decision

For apps that need native reminders or store distribution, wrap the existing web build with Capacitor and schedule reminders on the device with the local-notifications plugin. No server. The Android project is generated in CI and never committed. Testing runs in a CI emulator; release bundles are signed with a per-app upload key kept in repository secrets.

## Alternatives

- Keep a PWA only: rejected for reminders; the web has no dependable local scheduling.
- Server-side push: rejected for Taqweem because it needs a backend and breaks the local-first promise.
- A separate native codebase: rejected as higher cost than wrapping the web app.
- ICS export with alarms: kept as the honest path on the web only.

## Consequences

- Reminders work offline and send no data off the device.
- Vendor battery restrictions can still delay reminders on some phones; the emulator does not reproduce them.
- Google Play requirements apply (new personal accounts need a closed test of 12 testers for 14 days before production).
- Each app needs its own application id, keystores and store listing.

## Evidence

- Taqweem repository (`nawy-dev/Taqweem`): decisions D-161 to D-174, workflows `android.yml`, `android-emulator.yml`, `android-release.yml`.
- A CI emulator run on Android 14 and 15 observed the test and event notifications posted, and a Samsung A15 owner reported the test and a scheduled reminder arrived.
- Not verified: acceptance of `USE_EXACT_ALARM` by Google Play, long-term behavior after phone restarts, iOS.
