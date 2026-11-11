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
- any recorded verification (type-check, build, browser or payment runs).

If no specification exists, review against the request and the established behavior, and say so. Do not infer intended behavior from the implementation alone.

## Scale the review

For a small, bounded change, apply the checklists below yourself. For a substantial change (a new flow, a change to payments, persisted state or backend schemas, or several distinct areas), split the checklists across independent reviewers running in parallel, give each the exact comparison and its checklist, and merge their findings, removing duplicates.

## Requirements and behavior

- Requested outcomes missing, partial or wrong.
- Work added outside scope.
- Unintended changes to existing pages, URLs, wording or stored data.
- Claims of completion beyond the evidence.

## Patterns and simplicity

Use `file-organization`. Check for:

- a new dependency, context, machine, wrapper or abstraction without a concrete need;
- a parallel implementation of something that exists (a `src/ui` component, a `src/shared` helper, a data-access operation, an analytics helper);
- a backend `fetch` written in a component instead of `src/data-access`, or a response used without its validator;
- a domain-free helper kept privately beside a component instead of in `src/shared`;
- naming that breaks the conventions in `AGENTS.md`, or names that do not say what they hold;
- `.then()` chains, positional parameters in new functions, `export default` outside framework files;
- `"use client"` higher in the tree than needed, or browser APIs in a server component;
- comments that restate code or narrate the change, new commented-out code.

Similarity or file length alone is not a finding; name the failure or maintenance cost.

## Correctness and security

Apply every item of "Risk areas" and "Security and configuration" in `AGENTS.md` that the change touches. Look in particular for:

- an amount sent to a payment provider computed from the request body, wrong rounding to minor units, wrong currency;
- a secret in a `NEXT_PUBLIC_` variable, in frontend code, in a committed `.env` file, or in a log line or error response;
- personal data logged or exposed through a public endpoint; a new endpoint without input validation or with more public access than needed;
- a record read or changed by an ID from the request without checking that it belongs to the current user or that their role allows it;
- user input inserted into HTML without escaping (`dangerouslySetInnerHTML`, HTML e-mail templates, strings built into markup);
- a schema change without its matching fetch and validator changes, a "change together" list from `AGENTS.md` left incomplete, edits to generated files, an unstated effect on production data;
- a changed shape of persisted client state that breaks data users already have; storage read during server render;
- a form submitted twice, an error the user never sees, a missing success or failure path;
- missing metadata on a new page, a changed public URL, a broken feed or sitemap;
- renamed analytics events or payload fields, tracking that ignores consent.

## Everyday usability

For pages, forms and controls, read `frontend-design` and compare with the closest existing screen:

- reuse of `src/ui` components instead of raw equivalents;
- text in the UI language that matches the tone and terms of nearby screens; no text left in another language;
- labels on inputs, validation messages under the affected field, entered values kept after an error;
- loading, empty and error states; a busy button that blocks a second click;
- layout at mobile and desktop widths, built mobile first, overflow, keyboard focus;
- images with `alt` text and sizes that do not shift the layout.

## Verification

- Map each claimed outcome to the check that proves it (`AGENTS.md`, "Checks"; `implement`, "Build and verify").
- Type-check passes on the reviewed revision; a build for routing, metadata or configuration changes.
- A payment change was exercised end to end with test keys. A screenshot does not prove a submission or a redirect.

## Findings and report

Validate every finding against the surrounding code. Each actionable finding has:

- severity: **critical** (money, secrets, personal data, a broken core flow), **high** (common wrong behavior, lost function), **medium** (limited failure or real maintenance cost), **low** (narrow, concrete impact);
- file and tight line range;
- the concrete failing scenario;
- the rule, pattern or requirement it breaks;
- the smallest durable correction.

Order findings by severity. Separate actionable findings, verification gaps and optional suggestions. If there are no findings, say so, and list what was checked and what remains unverified.
