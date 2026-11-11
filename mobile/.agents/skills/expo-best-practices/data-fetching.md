# Data fetching

## The API client

- One client, `src/data-access/apiClient.ts`, sends every request: base URL from `EXPO_PUBLIC_API_URL`, JSON headers, the auth header from the session, a timeout through `AbortController` (mobile networks hang rather than fail), and one error type carrying the status and a user-safe message.
- Every response is parsed with its operation's Zod schema before anything reads it. A parse failure is an error the screen shows, and is reported to the error tracker with the operation name, never with personal data.
- Auth refresh happens in the client: on a 401, one refresh request runs (concurrent requests wait for it), the original request retries once, and a failed refresh signs the user out.
- Requests that create something the user pays for or cannot easily undo carry an idempotency key, so a retry after a dropped connection does not create it twice.

## TanStack Query

- Each operation exports a hook (`useOrders`, `useCreateOrder`). Query keys come from `src/data-access/queryKeys.ts` and contain every input the request depends on (`["orders", { status }]`).
- Server data stays in the query cache. Copying it into `useState` or a global store creates two sources of truth; derive from the query result instead (`select`, or a computation in render).
- Set `staleTime` per query from how fast the data really changes; the default of 0 refetches on every mount.
- A mutation invalidates the queries it changes (`queryClient.invalidateQueries({ queryKey })`) or updates them with the server's response. Optimistic updates only where agreed, with a rollback on error.
- Show cached data while refetching: `isPending` (no data yet) drives the full loading state, `isFetching` at most a small indicator or the pull-to-refresh spinner.
- Tests create a fresh `QueryClient` per test with `retry: false` (`mobile-testing`).

## App state and network

React Native does not have the browser events TanStack Query listens to. Wire them once in the root layout:

- `focusManager.setFocused` from `AppState` changes (`active` = focused), so stale queries refetch when the user returns to the app.
- `onlineManager.setEventListener` from the network state (`@react-native-community/netinfo` or `expo-network`), so queries pause offline and resume when the connection returns.
- A screen that should refresh every time it becomes visible calls `refetch` inside `useFocusEffect`, guarded so it does not refetch on the first mount twice.

## Offline

- Decide per feature what works offline: read-only from the cache (persist the query cache with the query persister and a `buster` tied to the app version), or queued writes (paused mutations that resume online). Both are persisted state: the risk area in `AGENTS.md` applies.
- An offline state tells the user what is happening and what will happen when the connection returns; it never looks like an empty list.
