---
name: code-review
description: "Use when reviewing a PR, branch, commit range, or uncommitted changes, or checking completed implementation before handoff."
---

# Code review

Review the requested change and the behavior it affects. Report defects introduced or exposed by it, not a repository-wide improvement list.

## Establish the comparison

Use the commit, branch or pull request the user names. For a branch without a stated base, use its merge base with the release branch named in `AGENTS.md`. For local work, include staged, unstaged and untracked files; a diff ending at `HEAD` misses uncommitted work. State what the review covers.

Read:

- `AGENTS.md` and the applicable local skills;
- the originating request or issue (`gh issue view`, `gh pr view`);
- every changed function or component in full, its callers and the closest comparable implementation;
- any recorded verification (type-check, tests, simulator and emulator runs, Maestro flows, sandbox purchases).

If no specification exists, review against the request and the established behavior, and say so. Do not infer intended behavior from the implementation alone.

## Scale the review

For a small, bounded change, apply the checklists below yourself. For a substantial change (a new flow, a change to purchases, auth, persisted state, native configuration or backend schemas, or several distinct areas), split the checklists across independent reviewers running in parallel, give each the exact comparison and its checklist, and merge their findings, removing duplicates.

## Requirements and behavior

- Requested outcomes missing, partial or wrong on either platform.
- Work added outside scope.
- Unintended changes to existing screens, routes, wording, permissions or stored data.
- Claims of completion beyond the evidence.

## Patterns and simplicity

Use `file-organization`. Check for:

- a new dependency, context, store, wrapper or abstraction without a concrete need; a native dependency where an installed package or an Expo module would do;
- a parallel implementation of something that exists (a `src/ui` component, a `src/shared` helper, a data-access operation, a storage wrapper, an analytics helper);
- a `fetch` written in a component instead of `src/data-access`, a response used without its validator, server data copied into local state instead of read from the query;
- a non-route file inside `src/app` (Expo Router turns it into a route);
- a domain-free helper kept privately beside a component instead of in `src/shared`;
- naming that breaks the conventions in `AGENTS.md`, or names that do not say what they hold;
- `.then()` chains, positional parameters in new functions, `export default` outside route and config files;
- colors, sizes or fonts written as literals instead of `src/theme` tokens;
- comments that restate code or narrate the change, new commented-out code.

Similarity or file length alone is not a finding; name the failure or maintenance cost.

## Correctness and security

Apply every item of "Risk areas" and "Security and configuration" in `AGENTS.md` that the change touches. Look in particular for:

- a native change (new native dependency, config plugin, permission, `app.config.ts` native field) described or shipped as an OTA update, or without a runtime version change;
- an API change that breaks app versions already installed;
- an entitlement or amount decided in the app, a purchase not verified on the server, a digital good sold outside in-app purchases;
- a secret in an `EXPO_PUBLIC_` variable, in `app.config.ts` `extra`, in app code, in a committed `.env` file, or in a log line;
- a token stored outside `expo-secure-store`; personal data logged or sent to analytics; sign-out that leaves user data on the device;
- a backend endpoint that reads or changes a record by an ID from the request without checking that it belongs to the current user or that their role allows it;
- user input inserted into HTML without escaping (WebView content, HTML e-mail templates sent by the backend);
- a changed shape of persisted state that breaks data users already have;
- deep link params or notification payloads used without validation, or opening a route the user may not see;
- a permission requested on launch instead of at the moment of use, a missing usage description, a denied permission that leaves the feature stuck;
- a schema change without its matching validator changes, a "change together" list from `AGENTS.md` left incomplete, edits to generated files or to `ios/` / `android/` in a CNG project;
- a form submitted twice, an error the user never sees, a missing success or failure path, a request with no timeout;
- effects, listeners, timers or subscriptions without cleanup; state updates after unmount; work repeated on every render of a list item;
- renamed analytics events or payload fields, tracking that ignores consent.

## Everyday usability

For screens, forms and controls, read `mobile-design` and compare with the closest existing screen:

- reuse of `src/ui` components instead of raw equivalents;
- text in the UI language that matches the tone and terms of nearby screens; no text left in another language;
- safe areas, the keyboard covering no focused input, the Android back button and the iOS swipe-back doing the expected thing;
- loading, empty, error and offline states; pull to refresh on lists; a busy button that blocks a second tap;
- long lists virtualized, images sized so nothing jumps;
- touch targets large enough, labels and roles for screen readers, layout surviving a large font size and dark mode.

## Verification

- Map each claimed outcome to the check that proves it (`AGENTS.md`, "Checks"; `implement`, "Build and verify").
- Type-check, lint and tests pass on the reviewed revision.
- UI changes were seen on both platforms. A purchase change was exercised in the sandbox. A screenshot does not prove a submission, a purchase or a deep link.

## Findings and report

Validate every finding against the surrounding code. Each actionable finding has:

- severity: **critical** (money, secrets, personal data, a broken core flow, a crash, an update that breaks installed apps), **high** (common wrong behavior, lost function, one platform broken), **medium** (limited failure or real maintenance cost), **low** (narrow, concrete impact);
- file and tight line range;
- the concrete failing scenario, with the platform when it matters;
- the rule, pattern or requirement it breaks;
- the smallest durable correction.

Order findings by severity. Separate actionable findings, verification gaps and optional suggestions. If there are no findings, say so, and list what was checked and what remains unverified.
