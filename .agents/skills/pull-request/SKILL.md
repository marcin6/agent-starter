---
name: pull-request
description: "Push the current feature branch and open a pull request with the project's description template."
disable-model-invocation: true
---

# Pull request

Invoking this skill is the developer's request to push the current branch and open a pull request. It does not cover committing: if the working tree has uncommitted changes, list them and ask whether to commit them first.

## Prepare

1. Read `AGENTS.md`, "Git" and "Deployment": the release branch the pull request targets and what merging into it deploys.
2. Confirm the current branch is a feature branch, not the integration or release branch. If it is not, stop and say so.
3. Read what the branch contains: `git log <release-branch>..HEAD --oneline` and `git diff <release-branch>...HEAD --stat`, then the diff itself. Read the linked issue or spec (`gh issue view`, `feature-spec`).
4. Run the checks from `AGENTS.md`, "Checks" that apply to the changed areas and have not run on this revision. If one fails, report it and stop; a pull request is opened on a green revision.

## Write the description

Title: the same style as a commit subject in `AGENTS.md` (short, lowercase, imperative).

```markdown
## Summary
<What changes for the user and why. "Closes #<issue>" when an issue exists.>

## Changes
- <2–5 bullets grouped by area: backend, data-access, UI, configuration>

## Risk areas
<Each item from `AGENTS.md` → "Risk areas" the change touches, and its effect. "None" if none.>

## Deployment notes
<New environment variables and whether they already exist in hosting; backend changes that must deploy first; schema changes and their effect on existing data. "None" if none.>

## Checks
- <Each command run, with its result>
- <Manual checks: pages and widths in the browser, flows walked through end to end>
- <What remains unverified>
```

Delete a section only when it truly has nothing to say; "None" under "Risk areas" and "Deployment notes" is information for the reviewer.

## Open it

1. Push the branch: `git push -u origin <branch>`.
2. Create the pull request with the body on stdin: `gh pr create --base <release-branch> --title "<title>" --body-file -`.
3. Report the pull request's URL and any check that could not be run.
