---
name: release
description: "Use when deciding whether a change can ship as an OTA update, or when changing version numbers, runtime versions, eas.json, store metadata, or preparing a build, submission or update."
---

# Release

A mobile release cannot be rolled back on users' devices the way a website can: a broken store build stays installed until users update, and a broken OTA update reaches every compatible build. Run `eas build`, `eas submit` and `eas update` only when the developer asks, and state exactly what will ship where before running them.

## JS-only or native?

Classify every change before it ships:

| Change | Kind | Ships as |
| --- | --- | --- |
| JS / TypeScript code, styles, bundled images and fonts, `EXPO_PUBLIC_*` values | JS-only | An OTA update to builds with the same runtime version, or the next build |
| A dependency with native code added, removed or upgraded; an Expo SDK upgrade | Native | A new build and a store release |
| A config plugin, permissions, `scheme`, associated domains, intent filters, icons, splash, bundle identifier / package | Native | A new build and a store release |
| JS that calls a native API or module only present in a newer build | Native in effect | A new build; an OTA update would crash older builds |

When unsure, treat it as native. A JS-only change that needs a backend change ships after the backend.

## Versions

- `version` (marketing version, `1.4.0`) is what users and the stores see. Raise it for every store release.
- Build numbers (`ios.buildNumber`, `android.versionCode`) must grow for every uploaded build. <They are managed by EAS (`appVersionSource: "remote"` and `autoIncrement` in `eas.json`) / set by hand in `app.config.ts`.>
- The runtime version decides which builds an update may reach. Policy: <`appVersion` (raise `version` for every native change) / `fingerprint` (computed from the native project, changes automatically with native changes)>. With `appVersion`, a native change without a new `version` lets an OTA update reach builds missing that native code: the app crashes.

## Build profiles (`eas.json`)

- `development`: the development client for engineers (internal distribution, simulator builds).
- `preview`: a production-like build for testers (internal distribution, TestFlight or an internal testing track), channel `preview`.
- `production`: the store build, channel `production`.

Each profile has its own EAS environment variables. A new variable is added to every profile that needs it before the build.

## OTA updates

1. Confirm the change is JS-only and which runtime version and channel it targets.
2. Publish to the `preview` channel first (`eas update --channel preview --message "<what changed>"`) and check it on a preview build of the same runtime version: restart the app (updates usually apply on the next launch) and walk through the changed flow.
3. Publish to `production` only when the developer asks. Upload source maps to the error tracker for the update.
4. If an update is broken, republish the last good update or roll back to the embedded build (check `eas update --help` for the commands in the installed EAS CLI). Never "fix forward" a crash in a hurry without checking it on `preview`.

Store rules allow OTA updates that fix bugs and change content; a change to the app's core purpose or a new paid feature goes through store review.

## Store builds and submission

1. All checks from `AGENTS.md` pass on the release commit; the critical Maestro flows pass on a `preview` build of it on both platforms.
2. `version` is raised; release notes are written in the store languages.
3. `eas build --profile production --platform all`, then `eas submit` (or `--auto-submit`).
4. Before submitting, check what store review checks: privacy labels (App Store) and the data safety form (Google Play) match the data the app collects; the privacy manifest covers the APIs used; permissions all have a reason; account deletion exists if accounts can be created; digital goods use in-app purchases; sign-in options meet the current App Store guidelines; demo account credentials are provided to reviewers.
5. Prefer a phased rollout (App Store phased release, Google Play staged rollout) and watch crash-free sessions before completing it.

## Old versions

- The backend keeps every API the oldest supported app version uses. Removing a field or endpoint waits until that version is no longer supported.
- <How the app enforces a minimum version: a version check on launch against <endpoint / remote config> that asks for or forces a store update.>

## Report

State what was built, submitted or published: profile, platform, version, build number, runtime version, channel, and the checks run on it. Name anything that still needs the developer (store forms, review answers, rollout decisions).
