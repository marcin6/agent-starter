# Debugging

Start from the exact error text and where it appears (Metro terminal, red screen, native crash, build log). Reproduce on the platform where it happens before changing code.

## Common errors and their usual cause

| Symptom | Usual cause | First step |
| --- | --- | --- |
| "Cannot find native module", "… has not been registered", crash on launch after adding a package | The installed development build does not contain the new native code | Rebuild the development build (`npx expo run:<platform>` or `eas build --profile development`) |
| Errors after installing or upgrading a package, duplicate React, odd type errors from libraries | A version not compatible with the SDK | `npx expo install --check`, then `npx expo-doctor` |
| Old code keeps running, strange bundling errors after switching branches | Stale Metro cache | `npx expo start --clear` |
| Native build fails after config changes | Generated projects out of date | `npx expo prebuild --clean`, then build again |
| Network request failed from the Android emulator to a local server | `localhost` is the emulator itself | Use `10.0.2.2` (Android emulator) or the machine's LAN IP (physical device); the server must listen on `0.0.0.0` |
| Plain `http://` requests fail | iOS ATS / Android cleartext rules | Use HTTPS; allow cleartext only for the local development host |
| Works in development, fails in a release build | Code relying on dev-only behavior, a missing `EXPO_PUBLIC_` variable in the EAS environment, or a minification issue | Reproduce with a release build locally (`--variant release` / `--configuration Release`) |
| Works on one platform only | Platform-specific API, permission, font name or style (shadows, `overflow`, text clipping) | Compare both platforms at the same commit |
| Update published but the app does not change | Different channel or runtime version, or the update is applied on the next launch | Check the build's channel and runtime version (`release`) and restart the app twice |

## Tools

- **Metro terminal**: JS logs and bundling errors. Press `j` to open React Native DevTools (console, sources, React components, profiler), `r` to reload, `m` for the dev menu.
- **iOS native logs**: Xcode's console for the simulator or device, or `xcrun simctl spawn booted log stream --predicate 'process == "<AppName>"'`.
- **Android native logs**: `adb logcat` (filter with `*:E` or the package name), or Android Studio's Logcat.
- **Network**: the Network tab in React Native DevTools where the installed version supports it, otherwise a proxy (<Proxyman / Charles>) or logging in `apiClient`.
- **Crashes in production**: <Sentry / the error tracker>, with source maps uploaded for every build and update.

## Discipline

- Change one thing at a time and keep the reproduction; a rebuild, a cache clear and a code change at once hide which one fixed it.
- A fix for a crash includes the cause in the report, not only the change that made it disappear.
