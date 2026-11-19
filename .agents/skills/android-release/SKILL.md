---
name: android-release
description: Use when wrapping a Nawy web app for Android, adding local notifications, testing in a CI emulator, signing, or publishing to Google Play.
---
# Android Release

Proven only in Taqweem (see `ANDROID-AND-PLAY.md`). Treat it as a verified route for that app, not as proof for another one.

## Workflow
1. Read `AGENTS.md`, `SECURITY-AND-APPROVALS.md`, `KNOWN-LIMITATIONS.md` and `ANDROID-AND-PLAY.md` in full. Inspect the target app's real code, `package.json`, build output and CI before assuming anything from Taqweem applies.
2. Settle `appId` and app name with the owner first: the id is permanent after the first Google Play upload.
3. Wrap the web build with Capacitor. Generate `android/` in CI with `npx cap add android`, do not commit it, and patch the manifest, icons and Gradle in CI (order and scripts are in the guide).
4. If the app has native reminders, follow the notification rules in the guide: never resolve a promise with the Capacitor plugin object, surface scheduling errors, verify pending items, and give new items a default reminder.
5. Test in a CI emulator (API 34 and 35) before giving the owner anything. Read results from the CI log. Do not ask the owner to try a build that the emulator has not passed.
6. Give the owner one install link and an in-app diagnostics screen showing the build version. State what the emulator cannot prove (vendor battery limits).
7. Create a fresh debug keystore and a fresh upload key per app. The owner adds the three signing secrets by hand; build a signed bundle only through a manual workflow that publishes a draft release.
8. Prepare store material (privacy page, data-safety answers, graphics, listing text, closed-test plan) and mark every unverified claim as Unknown.
9. Report what was tested with exact CI evidence, what was not verified, and what only the owner can do (developer account, forms, upload, testers).

## Hard rules
- Never commit keystores, passwords or secrets; never print them in logs.
- Do not claim Play acceptance, a published app, or a device result that was not observed.
- Do not upload, submit forms or change production on the owner's behalf without explicit authorization.
- Do not describe features in the store listing that the app does not have.
