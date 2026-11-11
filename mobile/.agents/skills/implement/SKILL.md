---
name: implement
description: "Use when implementing a feature, fixing a bug, refactoring behavior, or carrying out an approved specification or issue."
---

# Implement

Deliver the smallest complete change that satisfies the request, works on iOS and Android, and keeps every risk area in `AGENTS.md` intact.

## Establish the change

1. Read `AGENTS.md`, then the issue, spec or request (`gh issue view` when an issue is linked; `feature-spec` when the change has a spec).
2. Check the branch and working tree. Preserve unrelated user changes.
3. Read the current implementation, its callers and the closest working example: the neighbouring screen, component, data-access operation or backend controller.
4. Read `file-organization` for placement and `mobile-design` for UI changes. Consult `expo-best-practices` only for the decisions the change involves, against the installed SDK.

Before editing, briefly state:

- the observable outcome on each platform and what is out of scope;
- the existing code and UI pattern you will reuse;
- any new component, dependency, permission, storage key, route or endpoint and the concrete need for it;
- whether the change is **JS-only** (can ship as an OTA update) or **native** (needs a new build and a store release; see `release`);
- the checks that will prove the change (`AGENTS.md`, "Checks");
- every risk area from `AGENTS.md` the change touches, and its effect.

Settle routine choices from the request and the repository. Ask only when a decision materially changes what users see or pay, stored data, a route or deep link, a permission, a native dependency, or an external contract (stores, payment provider, push provider, third-party APIs).

## Keep the solution small

Extend the existing owner directly. Keep feature code in its feature folder; share it only when a second consumer needs it. A domain-free helper goes to `src/shared` from its first use.

Add a dependency, context, store, wrapper or cache only for a named requirement. A dependency with native code costs more than a JS one: a new development build, a store release, and an upgrade to carry through every SDK bump. Prefer an Expo module or a package already installed. Refactor inside the affected area only when it is needed for correctness or makes the requested change materially easier; report other ideas separately.

## Build and verify

Work in small steps and keep type-check, lint and tests green. For logic changes, write the failing Jest test first, then make it pass (`mobile-testing`):

| Change | Evidence |
| --- | --- |
| Pure helper or calculation | A Jest test in the folder's `__tests__` covering the cases and boundaries |
| Component or screen behavior | A React Native Testing Library test of what the user sees and does, including loading, empty and error states |
| UI | The changed screen on the iOS simulator and the Android emulator, compared with the neighbouring screens (`mobile-design`) |
| Route, params, deep link | The screen opened cold through its deep link (`npx uri-scheme open <url> --ios` / `--android`), with valid and invalid params |
| Dependency or `app.config.ts` | `npx expo install --check`, `npx expo-doctor`, and a fresh development build on both platforms |
| Persisted state | A test that old stored data still reads or is discarded safely, then a restart of the app with old data present |
| Purchase or payment flow | Tests of the changed logic, then the full flow in the store sandbox / with test keys, including success, failure and cancellation |
| Critical user flow | The Maestro flow that covers it, updated and passing |
| Backend schema or endpoint | The backend running locally, the change exercised through it, the app reading it, and the previous app version still working |

If a simulator, emulator, device or test mode cannot run, report that gap; never call unverified behavior verified.

## Finish

Review the final diff with `code-review`, including new files. Fix material findings and repeat the affected checks.

When the change makes `AGENTS.md`, a skill or a spec describe something that is no longer true (a path, a flow, a command, a "change together" list), update that text in the same change.

Report what changed, whether it is JS-only or native, the patterns reused, any exception, the checks run on each platform with their results, and remaining gaps. Commit, push, build, submit or publish an update only when the developer asks.
