# Routing and navigation (Expo Router)

## Files are routes

- Every file in `src/app` (or `app/`) is a route; its default export is the screen. Components, hooks, tests and helpers live outside it.
- `_layout.tsx` wraps its folder: it renders a navigator (`Stack`, `Tabs`, or a custom one) and holds providers or guards that apply to every screen below it.
- `index.tsx` is the folder's default route. `[id].tsx` is a dynamic segment, `[...rest].tsx` a catch-all, `+not-found.tsx` the fallback for unknown paths.
- `(group)/` organizes routes and gives them a shared layout without adding a URL segment: typically `(tabs)`, `(auth)`, `(app)`.
- Screen options (title, header buttons, presentation) belong in the layout's `<Stack.Screen name="…" options={…} />` when static, or in a `<Stack.Screen options={…} />` rendered inside the screen when they depend on its data.
- Modals and sheets are stack screens with `presentation: "modal"` (or `"formSheet"` where supported), not ad-hoc overlays, so the back button and gestures work.

## Navigating

- Use `<Link href=…>` for navigation the user taps and `router.push` / `router.replace` / `router.back` from `expo-router` in handlers. `replace` for flows the user must not return to (after sign-in, after a completed checkout).
- Build hrefs with the object form (`{ pathname: "/orders/[id]", params: { id } }`) so typed routes catch broken links when they are enabled.
- `router.dismissAll()` or `router.dismissTo()` (check the installed version) closes a modal flow; do not stack `back()` calls.

## Params are untrusted strings

- `useLocalSearchParams()` returns strings (or string arrays for repeated keys), whatever the type parameter says. Parse them with a Zod schema at the top of the screen and render the not-found or error state for invalid values.
- `useLocalSearchParams` reads the params of this screen; `useGlobalSearchParams` re-renders on every navigation anywhere: use it only when the screen truly needs global params.
- Pass IDs in params and load the entity on the screen; never pass whole objects through params.

## Deep links and cold starts

- Every route is a deep link (`<scheme>://orders/42`) and can be the first screen the app opens. A screen therefore reads its own data from its params and does not depend on state left by the previous screen.
- A screen opened cold has no history. Provide a way back (a header button to the tab root, or an `unstable_settings` / anchor route in the layout, depending on the installed version) so the user is not stranded.
- Protected areas use the router's guard (`Stack.Protected guard={isSignedIn}` in recent versions) or a `<Redirect>` in the group's `_layout.tsx`. The guard waits for the session to load from storage before deciding, so a signed-in user does not flash the sign-in screen.
- Renaming or moving a route changes its deep link and breaks notification payloads, e-mails and universal links that point to it. Confirm the change with the developer; keep a redirect when the old path is in use.
- Test a link with `npx uri-scheme open "<scheme>://path" --ios` (or `--android`), both while the app runs and from a killed app.

## Screen lifecycle

- Screens in a stack stay mounted underneath the screen on top, and tab screens stay mounted after the first visit. Work that should run each time the screen becomes visible uses `useFocusEffect` with a cleanup, not `useEffect`.
- Timers, listeners and subscriptions started in a screen are cleaned up when it blurs or unmounts.
- Heavy screens deep in a stack keep memory; avoid keeping large lists of images in screens the user has left behind for long flows.
