# Native code and configuration

## Expo Go vs. development builds

- Expo Go contains a fixed set of native modules. It is enough only while the project uses nothing outside that set; it never reflects the production app's config (icons, permissions, plugins).
- A development build (`expo-dev-client`) is the project's own app with its native modules and config. Build it locally (`npx expo run:ios`, `npx expo run:android`) or with EAS (`eas build --profile development`). After that, `npx expo start` serves JS changes to it instantly.
- A new or upgraded dependency with native code, a new config plugin, or a change to a native field of `app.config.ts` means a new development build. Until it is rebuilt, the app crashes or reports a missing native module.

## Continuous Native Generation (CNG)

- `ios/` and `android/` are generated from `app.config.ts` and config plugins by `npx expo prebuild`. They are build output: not committed, never edited by hand. `npx expo prebuild --clean` regenerates them from scratch.
- A native change no library's plugin covers goes into a config plugin in `plugins/` (modifying Info.plist, AndroidManifest, Gradle or Podfile through the `expo/config-plugins` mods), registered in `app.config.ts`.
- A library that needs manual native setup and ships no config plugin is a cost the developer must agree to: it needs a project plugin to stay CNG-compatible.

## `app.config.ts`

- One config for every environment; values that differ (app name suffix, bundle identifier / package for side-by-side dev and prod installs, API URL, icons) come from environment variables (`APP_VARIANT`, EAS env vars).
- `ios.bundleIdentifier` and `android.package` are permanent once the app is in a store: changing them creates a different app.
- `scheme` defines the deep link prefix. Universal links (iOS `associatedDomains`) and Android app links (`intentFilters` with `autoVerify`) also need files served from the domain; changing any of them is a native change.
- `version` is what users see; build numbers and the runtime version follow the rules in `release`.

## Environment variables

- `EXPO_PUBLIC_*` variables are inlined into the JS bundle when it is built or updated: they are public, and changing one needs a new update or build. Read them as `process.env.EXPO_PUBLIC_NAME` written out in full; dynamic access (`process.env[name]`) is not inlined.
- Variables without the prefix are available to `app.config.ts` and build tooling only.
- For EAS builds and updates, variables come from EAS environments (`eas env:*`), not from the local `.env`. A local build and an EAS build can therefore differ: check both when a variable changes.

## Permissions

- Each permission comes from the library's config plugin with a usage description in the UI language that says why the app needs it ("to scan the barcode on your card"), set in `app.config.ts`. A missing iOS usage string crashes the app when the permission is requested and gets the build rejected.
- Request at the moment the feature needs it, after the user's action, never on launch. Explain first when the system prompt alone would confuse.
- Handle every outcome: granted, denied (can ask again on Android), and blocked (only `Linking.openSettings()` helps). The feature shows what it can do without the permission.
- Remove permissions a dependency adds that the app does not use (`android.blockedPermissions`); unused permissions draw store review questions.

## Platform-specific code

- Small differences: `Platform.OS` checks or `Platform.select({ ios, android })`.
- Large differences: `Component.ios.tsx` and `Component.android.tsx` with the same exports and props.
- Version checks (`Platform.Version`) only for a documented API difference, with the reason in a comment.
- <If the app also targets web: `.web.tsx` files and `Platform.OS === "web"`; web has no native modules, so every native feature needs a web fallback or a guard.>
