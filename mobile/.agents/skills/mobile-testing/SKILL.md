---
name: mobile-testing
description: "Use when writing, fixing, or running Jest unit tests, React Native Testing Library component tests, or Maestro end-to-end flows."
---

# Mobile testing

Test what the user sees and does, at the cheapest level that proves it. `AGENTS.md`, "Checks", says which changes require a test; `implement` says which evidence each change needs.

## Pick the level

| Question | Level | Tool |
| --- | --- | --- |
| Is this calculation, parser, reducer or storage migration right? | Unit | Jest |
| Does this component or screen show the right states and react to taps and input? | Component | Jest + React Native Testing Library |
| Does the whole flow work in the real app on a device? | End to end | Maestro on a simulator or emulator |

Most tests are unit and component tests. End-to-end flows cover only the critical paths (<sign-in, purchase, the core flow of the app>): they are slow and need a build.

## Jest setup

- The preset is `jest-expo`, which mocks Expo's native modules. A third-party native module without its own Jest mock gets one in the shared setup file (<`jest.setup.ts`>), once, not per test.
- Tests live in `__tests__` beside the code (`src/shared/__tests__/formatPrice.test.ts`), never in `src/app`.
- Run one file while working (`npx jest src/shared/__tests__/formatPrice.test.ts`), the whole suite before finishing (`npm test`).

## Component tests

- Render with the providers the component needs through one shared helper (<`renderWithProviders`> in `<path>`): a fresh `QueryClient` per test with `retry: false`, the theme, the safe area provider with fixed metrics.
- Query like a user: `screen.getByRole("button", { name: "Save" })`, `getByText`, `getByLabelText`. `testID` only when nothing user-visible identifies the element.
- Interact with `userEvent` (`const user = userEvent.setup(); await user.press(button); await user.type(input, "…")`); it simulates the full event sequence, unlike `fireEvent`.
- Wait for async results with `findBy…` or `waitFor`; never with fixed timeouts.
- Mock at the data-access boundary (the operation function or `apiClient`), not inside components, and cover the loading, empty, error and success states.
- Expo Router screens: render them with the router's testing utilities (`renderRouter` from `expo-router/testing-library`) and a mock file tree, then assert on the path and the rendered screen.
- Avoid snapshot tests of whole screens; they break on every style change and prove nothing about behavior.

## Logic that needs a test with every change

- Money: prices, totals, discounts, currency formatting.
- Validation: Zod schemas for API responses, forms, params, payloads.
- Persisted state: a test per storage version that old data migrates or is discarded safely.
- Deep link and notification parsing: valid, invalid and malicious input.
- Auth: token refresh, a failed refresh, sign-out clearing everything.

## End-to-end with Maestro

- Flows live in `.maestro/` as YAML, one per user flow, named after it (`sign-in.yaml`). Shared steps go into subflows (`runFlow`).
- They run against an installed development or preview build on a booted simulator or emulator: `maestro test .maestro/` (or one file). Expo dev builds need the flow to open the app at the right URL or launch the bundled build.
- Select elements by visible text or accessibility label first, `id` (testID) when text is not stable. Wait with `assertVisible` / `extendedWaitUntil`, not fixed sleeps.
- Use test accounts and sandbox purchases; a flow never touches production data.
- Run each changed flow on both platforms when the flow differs between them.

## Report

Name the tests added or changed, the commands run and their results. If a level could not run (no simulator, no build), say so and what remains unproven.
