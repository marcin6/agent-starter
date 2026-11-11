---
name: implement
description: "Use when implementing a feature, fixing a bug, refactoring behavior, or carrying out an approved specification or issue."
---

# Implement

Deliver the smallest complete change that satisfies the request and keeps every risk area in `AGENTS.md` intact.

## Establish the change

1. Read `AGENTS.md`, then the issue, spec or request (`gh issue view` when an issue is linked; `feature-spec` when the change has a spec).
2. Check the branch and working tree. Preserve unrelated user changes.
3. Read the current implementation, its callers and the closest working example: the neighbouring page, component, data-access operation or backend controller.
4. Read `file-organization` for placement and `frontend-design` for UI changes. Consult `next-best-practices` only for the Next.js decisions the change involves, against the installed version.

Before editing, briefly state:

- the observable outcome and what is out of scope;
- the existing code and UI pattern you will reuse;
- any new component, dependency, schema field or endpoint and the concrete need for it;
- the checks that will prove the change (`AGENTS.md`, "Checks");
- every risk area from `AGENTS.md` the change touches, and its effect.

Settle routine choices from the request and the repository. Ask only when a decision materially changes what users see or pay, stored data, a public URL or an external contract (payment provider, search engines, third-party APIs).

## Keep the solution small

Extend the existing owner directly. Keep feature code in its feature folder; share it only when a second consumer needs it. A domain-free helper goes to `src/shared` from its first use.

Add a dependency, context, state machine, wrapper or cache only for a named requirement. Refactor inside the affected area only when it is needed for correctness or makes the requested change materially easier; report other ideas separately.

## Build and verify

Work in small steps and keep type-check, lint and tests green. For logic changes, write the failing Vitest test first, then make it pass:

| Change | Evidence |
| --- | --- |
| Pure helper or calculation | A Vitest test in the folder's `__tests__` covering the cases and boundaries |
| Page, routing, metadata, server/client split | `npm run build` and the page in `npm run dev` |
| UI | The changed page in the browser at mobile and desktop widths, compared with the neighbouring screens (`frontend-design`) |
| Payment or other money flow | Tests of the changed logic, then the full flow locally against the provider's test keys, including the success and failure paths |
| Persisted client state | A test that old stored data still reads or is discarded safely, then a reload in the browser |
| Backend schema or endpoint | The backend running locally, the change exercised through it, and the frontend reading it |

If the app or a test mode cannot run, report that gap; never call unverified behavior verified.

## Finish

Review the final diff with `code-review`, including new files. Fix material findings and repeat the affected checks.

When the change makes `AGENTS.md`, a skill or a spec describe something that is no longer true (a path, a flow, a command, a "change together" list), update that text in the same change.

Report what changed, the patterns reused, any exception, the checks run with their results, and remaining gaps. Commit, push or open a pull request only when the developer asks.
