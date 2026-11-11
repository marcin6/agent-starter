# Storage and security

## Pick the store by what the data is

| Data | Store | Why |
| --- | --- | --- |
| Tokens, credentials, keys | `expo-secure-store` | Keychain / Keystore encryption. Small values only; it is slow for frequent reads, so cache the token in memory after the first read. |
| Preferences, flags, small UI state | <MMKV / AsyncStorage> | Fast key-value storage. Unencrypted: readable on a rooted or backed-up device. |
| Structured or large offline data | `expo-sqlite` | Queries, transactions, migrations. |
| Downloaded files, images | `expo-file-system` (cache or document directory) | The cache directory can be cleared by the system; the document directory persists. |

## Every stored value is versioned and validated

- Each key has a wrapper in `src/storage` with a Zod schema and a version (`orders.filters.v2`). The wrapper is the only code that reads or writes the key.
- Reading parses the stored value: valid → use it; an older version → migrate it; invalid or unknown → discard it and fall back to the default. A read never throws into the UI.
- SQLite schema changes are migrations run in order on startup, each in a transaction, tested against a database created by the previous version.
- Stored data survives app updates and, through device backups, even reinstalls (iOS Keychain items can survive uninstall). Code reading storage assumes it may have been written by any older version.

## Sign-out and account deletion

- Sign-out removes tokens, clears the query cache and the persisted query cache, deletes every user-specific key, file and table, unregisters the push token on the server, and resets analytics identity. List what it clears in the storage wrappers so a new key is not forgotten.
- Account deletion (required by the App Store when the app lets users create accounts) deletes the server account and then everything sign-out clears.

## The app is public

- Everything in the binary can be read: the JS bundle, `EXPO_PUBLIC_*` values, `app.config.ts` `extra`, bundled assets. API keys shipped in the app are public keys: restrict them on the provider's side (bundle ID, package name, allowed APIs, quotas).
- Business rules that protect money or data run on the server. The app's checks are for the user's convenience only.
- Data received from deep links, push payloads, the clipboard, shared files and WebViews is untrusted: validate it before use, and never pass it into a URL the app then loads with the user's token.
- Logs and error reports carry IDs and operation names, never tokens, passwords or personal data.
- Screens with sensitive data (<payments, health, documents>) may hide content in the app switcher; agree on it with the developer.
