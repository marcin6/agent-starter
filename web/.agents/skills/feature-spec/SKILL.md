---
name: feature-spec
description: "Use when asked to write a spec, design document or implementation plan for a change spanning several areas, steps or sessions, or to resume work from one."
---

# Feature spec

A spec settles a larger change before code is written and carries it across sessions. It records the agreed behavior, the decisions and their reasons, and the progress; the code stays the source of truth for how things work.

Write one when a change touches several areas (data shape, backend, UI), has edge cases worth agreeing on, or will take more than one session. A small change goes straight to `implement`. Open questions are settled first, with `grilling` when there are many. A single unit of work for someone else is a GitHub issue (`github-issues`).

## Write it

1. Read `AGENTS.md`, the code involved and the closest existing feature. Facts that can be looked up are your work; ask the developer only for decisions.
2. Save the spec where the developer says: by default `docs/specs/<kebab-case-name>.md`, or the body of the issue it belongs to.
3. Fill in the template below. Name code by path and symbol (`src/data-access/fetchOrders`, `calculateTotal`), never by line number: line numbers go stale with the first edit.
4. Show the spec to the developer and wait for their approval before implementing.

## Template

```markdown
# <Feature name>

## Goal
<What the user can do afterwards and why it matters. One paragraph.>

## Out of scope
<What this change deliberately leaves out, and likely follow-ups.>

## Decisions
- <Decision> — <reason>; <rejected alternative and why>.

## Files touched
### <Area: backend / data-access / UI / config>
- `<path>` — <what changes>.

## Data flow
1. <From the user's action, through each layer, to the stored result and what the user sees.>

## Edge cases
| Case | Behavior |
| --- | --- |
| <Invalid or tampered input> | <What happens> |
| <Existing data created before this change> | <What happens> |
| <Double submit, retry, concurrent change> | <What happens> |
| <An external service fails or is slow> | <What happens> |

## Risk areas
<Each item from `AGENTS.md` → "Risk areas" this change touches, and its effect. "None" if none.>

## Steps
- [ ] 1. <A small step that leaves the app working> — check: <the command or manual check that proves it>.
- [ ] 2. …

## Testing plan
<Tests to add per step, manual checks at mobile and desktop widths, flows to walk through end to end.>

## Rollout and rollback
- Existing data: <how rows or stored state created before the change behave; whether a migration is needed>.
- Configuration: <new environment variables and every environment they must exist in before deployment>.
- Order: <what deploys first, e.g. the backend before the frontend that reads it>.
- Limiting exposure: <for a risky change, how it reaches a few users or runs in a watch-only mode first, and what to watch>.
- Rollback: <what to revert, and whether data written in the meantime is harmless>.
```

Delete a section that does not apply instead of writing "N/A" in it.

## Implement from it

- Work through the steps in order with `implement`. After a step passes its check, tick it and add what verified it (`- [x] 2. … — verified: tests pass, form checked at 375 px`).
- When the implementation departs from the spec, update the spec in the same change: the new decision and its reason.
- Resuming in a new session: read the spec, confirm with `git log` and the code that the ticked steps are really done, then continue from the first unticked step.

## Finish

When every step is ticked, move what stays true beyond this feature (a reason, a "change together" list, a new risk) into `AGENTS.md` or the skill it belongs to. Then delete the spec or keep it as a record, as the developer prefers. A kept spec must not contradict the code; mark it as completed at the top so nobody treats it as current instructions.
