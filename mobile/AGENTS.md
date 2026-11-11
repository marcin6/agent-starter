<!--
TEMPLATE. Replace every <placeholder>, delete every section or line that does not apply, then delete this comment and the other HTML comments (they load into the agent's context on every turn).
Keep only what an agent cannot find by reading the code: unwritten conventions, the reasons behind choices, hazards, lists of places that must change together. Commands and versions the agent can read from package.json, app.config.ts or eas.json do not belong here.
-->

# <Project name>

<One paragraph: what the app is, who uses it, the language of its UI, the platforms it ships to (iOS, Android, web or not), whether it sells anything and through what (in-app purchases, an external payment provider).> The repository holds <the layout, e.g. "a single Expo app" or "two independent npm projects: `mobile/` and `backend/`">:

- `<mobile/ or root>`: Expo SDK <version> with Expo Router, React Native <version>, TypeScript, <styling: `StyleSheet` with tokens from `src/theme` / NativeWind>, TanStack Query for server state, Zod for validation. Built and released with EAS. Node <version>.
- `<backend/>`: <framework, API, database locally and in production, hosting>. Node <version>.

Verify before answering or changing anything: read this file, the code involved and its callers. The user's claims and the existing code are evidence to check, not proof. If a request conflicts with this file, say so and ask for confirmation before proceeding.

## Local skills

Skills live in `.agents/skills/` (`.claude/skills` links to it). Read the applicable one before working in its area: `implement` for any feature, fix or refactor; `code-review` for reviewing changes; `file-organization` before creating, moving or naming files; `mobile-design` for any screen, component or interaction; `expo-best-practices` for routing, data fetching, lists, native configuration, storage and debugging; `mobile-testing` for writing or running tests; `release` for builds, version numbers, OTA updates and store submissions. Before using or changing a framework API, check the documentation for the installed version (the Expo SDK version in `package.json`, docs.expo.dev for that SDK), because SDK versions differ. Generic library advice applies only to the installed version and never justifies a new dependency, cache, abstraction or unrelated refactor on its own.

## Golden rules

- Keep things simple. YAGNI unless told otherwise. <Who maintains the project, e.g. "a small product maintained by one or two developers">: minimize concepts, moving parts and what someone must know to change the code safely.
- Implement the smallest complete solution for the agreed behavior. Add flexibility only for a current requirement or an existing consumer.
- Investigate before changing anything. Search for an existing solution (component in `src/ui`, helper in `src/shared`, data-access operation, hook) before creating a new one.
- Reuse sound patterns. Existing patterns are not automatically correct: if one is brittle, unsafe or needlessly complex, explain the problem and prefer the simplest durable correction within scope. Keep unrelated improvements out of scope; report them instead.
- Use type safety: strict TypeScript, a runtime validator (Zod) for every response from the backend or an external API and for every value read from storage, deep links or push notifications, typed props.
- Treat iOS and Android as two products sharing code: every change works and is checked on both.
- Propose bold ideas when they would meaningfully help (a new library, a test setup, a restructuring), but propose them to the developer instead of implementing them.
- Be objective: verify claims, state trade-offs and blind spots, recommend the strongest option plainly.
- Before declaring work complete, verify it in proportion to its risk (see "Checks"). Never claim success without evidence.

## How the app works

<!-- One short paragraph per domain area: its source of truth, its flow, and what must change together. Write the reason when a choice looks odd. The paragraphs below are defaults for an Expo Router app over a separate API; adapt or delete them. -->

**Data.** <The source of truth for each kind of data, e.g. "the API holds accounts, orders and the catalog; the device holds only preferences and a cache">. API types come from <the Zod schemas in `src/data-access/*/validators.ts` / a generated client in <path>, never edited by hand>.

**Reading and writing data.** Screens reach the API only through `src/data-access/<operation>/`: a function that calls the shared API client (`src/data-access/apiClient.ts`, which adds the base URL from `<EXPO_PUBLIC_API_URL>`, the auth header and a timeout) and parses the response with the Zod schema in `validators.ts` beside it, plus a TanStack Query hook (`use<Operation>`) with its query key. Fields the API does not guarantee are `nullable`; a list skips and logs an invalid entry instead of failing the whole screen. Mutations invalidate the queries they change. Queries refetch when the app returns to the foreground and when the network comes back.

**Auth.** <How the user signs in, where the session lives (tokens in `expo-secure-store`, never in AsyncStorage), how the token is refreshed (one refresh at a time; requests waiting on it retry once), what sign-out clears (tokens, the query cache, every user-specific key in storage), which routes are protected and how (`Stack.Protected` / a redirect in the group's `_layout.tsx`).>

**Client state.** <What is global (a context with a pure reducer / a Zustand store in `src/state`), what stays screen-local, what persists on the device, in which store (MMKV / AsyncStorage / SQLite) and under which key and version.> Persisted state is read through the typed wrappers in `src/storage`, which validate it and migrate or discard old shapes.

**Navigation and deep links.** Every route in `src/app` is reachable by a deep link (`<scheme>://`) <and a universal/app link on `<domain>`>, so every screen can open cold: it reads its params, validates them and loads its own data. <Push notification payloads and links in e-mails point to these routes: list them.> Renaming a route breaks those links; confirm it with the developer first.

**Purchases and payments.** <The flow: what is sold, through what (in-app purchases via <library> for digital goods and subscriptions; <provider> for physical goods or services), which endpoint confirms the purchase, where the user lands after success and failure.> **Every entitlement and every amount charged is decided on the server from stored data and verified receipts, never trusted from the app.** Prices in the app are display only.

**Push notifications.** <Provider (expo-notifications with Expo push service / FCM and APNs directly), where the device token is registered and removed, which payload types exist and the route each one opens.>

**Analytics and consent.** <Analytics tool, whether tracking waits for consent (and App Tracking Transparency on iOS), where event helpers live.> Send new events through those helpers only.

**Change together.** <Lists of places that must change in one change, e.g. "a new notification type: the backend sender, the payload Zod schema in `src/notifications`, the route it opens, the Maestro flow that covers it". Adding a field to an entity likewise means the backend schema, the Zod schema and the screens that show it.>

## Risk areas

`implement` and `code-review` check every item the change touches:

- **Native vs. JS changes.** A change to a dependency with native code, a config plugin, or a native field of `app.config.ts` (permissions, scheme, bundle identifier, icons, splash) needs a new development build and a new store release; it cannot ship as an OTA update. State which kind every change is (`release`).
- **Old app versions.** Users run old builds for months. An API change stays compatible with every app version still supported (<minimum supported version and how the app enforces it>), and an OTA update reaches only builds with the same runtime version.
- **Money.** Entitlements and amounts charged come only from the server. Digital goods and subscriptions are sold through the stores' in-app purchase systems as their rules require; receipts are verified on the server.
- **Persisted state.** Data in MMKV, AsyncStorage, SQLite or SecureStore outlives an update: a changed shape still reads old data or discards it safely, and sign-out removes everything user-specific.
- **Secrets and the bundle.** Everything in the app binary is public: `EXPO_PUBLIC_*` variables, `app.config.ts` `extra`, and every string in the JS bundle can be extracted. Secrets stay on the server.
- **Permissions and privacy.** Each permission has a usage description, is requested at the moment the feature needs it, and the feature handles denial. New data collection updates the App Store privacy labels and the Google Play data safety form <and the privacy manifest>.
- **Deep links and notifications.** Params and payloads are untrusted input: validated, and leading only to routes the user may see.
- **Platform parity.** Behavior differs between iOS and Android (back button, keyboard, permissions, date pickers, fonts, shadows): both are checked.
- <Project-specific risk, e.g. "**Offline orders.** An order created offline is sent once, with an idempotency key.">

## Security and configuration

- `EXPO_PUBLIC_*` variables are inlined into the JS bundle at build time. Only public values carry that prefix (API base URL, public keys). Build-time secrets (EAS tokens, Sentry auth token) live in EAS environment variables and are read only by build tooling.
- `.env` files hold real values: never print them, never commit one, and add new variables to `.env.example` with placeholder values.
- Tokens and other credentials live in `expo-secure-store`. Personal data (e-mail, address, phone, location) is never logged and never sent to analytics.
- The backend validates every input with a schema, answers errors without internal details, rate limits public write endpoints, and grants anonymous access per endpoint only as far as the product needs it.
- Network traffic uses HTTPS only. A WebView loads only the app's own origins.
- Production data lives on <host>. Before a schema change or a backend upgrade, state the effect on existing data and on installed app versions, confirm with the developer and make sure a backup exists (<backup command or "ask the developer">).

## Code conventions

<!-- Defaults; change them before the first line of code and keep them afterwards. -->

Follow `file-organization` for placement and `mobile-design` for UI. Project-wide:

- Imports across folders use the `@/` alias (`@/data-access/fetchOrders`, `@/ui`, `@/shared`); relative imports only inside one folder.
- Named exports. `export default` only where the framework reads it (route files and `_layout.tsx` in `src/app`) and in config files.
- Naming: interfaces `I` + PascalCase (`IOrder`, a component's props as `I{Component}Props`), type aliases `T` + PascalCase (`TOrderStatus`), Zod schemas `{Name}Schema`, module constants holding a fixed value `UPPER_SNAKE_CASE`. Names say what they hold, spelled out (`deliveryPrice`, not `res` or `data` where a domain name exists); booleans read as questions (`isLoading`, `hasDiscount`).
- New functions take one destructured, typed object (`fetchOrder({ id })`); callbacks and signatures dictated by a framework keep their shape.
- Asynchronous code uses `async`/`await` with `try`/`catch`, never `.then()` chains. A failure the user should know about is shown in the UI, not only logged.
- Styles use `StyleSheet.create` at the bottom of the component file with values from `src/theme`; platform differences use `Platform.select` or, for a large difference, `.ios.tsx` / `.android.tsx` files.
- Comments are in English, and only where a reader would otherwise be misled: a non-obvious reason, an external constraint, or a hazard. No comments that narrate the change or restate the code; no new commented-out code.
- User-facing text is <language>, <"written directly in the components (no translation layer)" or "kept in <i18n setup>">. Keep the established tone and wording of nearby screens.
- Prices, dates and numbers are formatted with the shared helpers in `@/shared` (<locale>).

## Checks

<!-- Put the real script names here once package.json exists. -->

`npx expo start` serves the JS to a development build (`npx expo run:ios` / `npx expo run:android` builds one locally; <Expo Go is / is not enough for this project>). Unit and component tests use Jest with `jest-expo` and React Native Testing Library (`npm test`; tests in `__tests__` folders beside the code); end-to-end flows use Maestro (`.maestro/`). Logic where a mistake costs money or blocks users (prices, purchases, auth, input validation, persisted state, deep link parsing) gets a test with every change. Lint passes with no errors or warnings, and every change keeps it that way.

- Every change: `npx tsc --noEmit`, `npx expo lint` and `npm test` pass.
- A change to dependencies or `app.config.ts`: `npx expo install --check` and `npx expo-doctor` pass, and a fresh development build starts on both platforms.
- UI changes: the changed screen on the iOS simulator and the Android emulator, at a small phone size, in light and dark mode and with a large font size (`mobile-design`).
- A purchase or payment change is verified end to end in the stores' sandbox / test tracks and with the provider's test keys, never with live payments.
- Use npm and the existing `package-lock.json`. Install a new dependency only after the developer agrees to it, with `npx expo install <package>` so the version matches the SDK. Install and upgrade only versions released at least two weeks ago; a fix for a critical, published vulnerability may be newer, and says so in the pull request. Check `npm audit` when upgrading.

## Deployment

- Builds and releases go through EAS (`eas.json` profiles: <`development`, `preview`, `production`>). <Which branch or event starts which build: e.g. "a merge to `main` builds `preview` for internal testers; store releases are started by hand">. Run `eas build`, `eas submit` and `eas update` only when the developer asks; follow `release`.
- Production environment variables live in EAS (<"`eas env`" / project settings on expo.dev>), not in the repository. A change that reads a new variable is merged only after the variable exists there for every profile that needs it; say so in the pull request.
- <Backend hosting.> Its deployment is separate from the app's; deploy backend changes the app depends on first, keep them compatible with app versions already in the stores, and ask the developer before relying on a backend change being live.

## Git

- Every change starts from `<integration branch>`: update it and create a kebab-case feature branch from it (`add-order-filters`).
- Pull requests go from the feature branch to `<release branch>`.
- Commit, push, open pull requests or merge only when the developer asks.
- Commit messages: a short, lowercase, imperative subject without a type prefix (`add order filters`).
- `ios/` and `android/` are generated by `npx expo prebuild` and are not committed; native changes go through `app.config.ts` and config plugins. <Delete this line if the project commits native folders, and say why.>
