---
name: expo-best-practices
description: "Use when writing or reviewing Expo Router routes and layouts, data fetching, lists and performance, app config, native modules, permissions, device storage, or when debugging a build or runtime error."
user-invocable: false
---

## Project application

Read `AGENTS.md` and the nearest existing screen first. Apply these guides only to the Expo SDK, React Native and library versions installed in the project (`package.json`). Check an API against the documentation for that SDK (docs.expo.dev, with the SDK version selected; reactnative.dev for the matching React Native version); where it disagrees with a guide below, the documentation wins. Screens reach the API through `src/data-access` as `AGENTS.md` describes. A generic recommendation is not a requirement to add a dependency, cache layer or refactor; require a concrete benefit within scope. Where `AGENTS.md` or `mobile-design` makes a deliberate exception, the exception wins. For UI work also use `mobile-design`.

# Expo and React Native best practices

Read only the guide the change needs.

## Routing and navigation

See [routing.md](./routing.md) for:

- Expo Router files, layouts, groups, stacks, tabs and modals
- Params as untrusted strings, typed routes
- Deep links, cold starts and protected routes
- Screen lifecycle: focus, blur and what stays mounted

## Data fetching

See [data-fetching.md](./data-fetching.md) for:

- The API client, timeouts and auth refresh
- TanStack Query keys, stale times and invalidation
- Foreground and network-aware refetching, offline behavior

## Lists and performance

See [performance.md](./performance.md) for:

- Virtualized lists and list item rules
- Images, re-renders and the UI thread
- Startup time and measuring in a release build

## Native code and configuration

See [native-config.md](./native-config.md) for:

- Expo Go, development builds and Continuous Native Generation
- `app.config.ts`, config plugins and environment variables
- Permissions and platform-specific code

## Storage and security

See [storage-security.md](./storage-security.md) for:

- Choosing SecureStore, MMKV / AsyncStorage or SQLite
- Versioned, validated storage and sign-out cleanup
- What an attacker can read from the app

## Debugging

See [debugging.md](./debugging.md) for:

- Metro, cache and version-mismatch errors
- "Native module not found" and build failures
- Logs and dev tools on each platform, networking from simulators
