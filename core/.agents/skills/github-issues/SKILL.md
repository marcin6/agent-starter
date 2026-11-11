---
name: github-issues
description: "Use when creating, editing, labeling, closing, or checking duplicates and scope for individual GitHub issues."
---

# GitHub Issues

Make each issue understandable without chat history or private notes. Give enough context to explain the problem and expected outcome, then link supporting sources when useful.

An issue defines **what needs to change**, **why it matters**, confirmed constraints, and observable completion. Leave the final implementation to the assignee unless the repository or an existing decision already requires a specific approach.

## Core Rules

- Read relevant repository documentation, implementation, tests, and existing issues before writing.
- Treat reported behavior and proposed solutions as claims to verify, not facts.
- Search for duplicates and overlapping work before creating an issue.
- Do not create a separate issue when updating or consolidating existing work produces a clearer backlog.
- Use plain English. Be direct, concise, and specific. Avoid jargon, buzzwords, filler, and repeated context.
- Include detail when it changes scope, protects an important constraint, or helps verify the outcome.
- Separate confirmed facts from assumptions and open questions.
- Explain why each related issue, document, or file matters. Do not rely on links to explain the task.
- Add appropriate existing labels to every issue created or updated.

## Workflow

### 1. Understand and Investigate

Identify the requested outcome, its practical value, known constraints, and the repository or tracker in scope. Research the relevant domain before proposing how the work should be done.

Challenge the requested boundary:

- Is this an independently useful outcome or part of existing work?
- Would completing another issue make it unnecessary?
- Can it be assigned, prioritized, and verified independently?
- Will it still make sense if implementation is delayed?

If the boundary is weak, recommend a clearer one instead of creating the issue as requested.

### 2. Check Existing Issues

Search open issues using relevant business terms, systems, and symptoms. Search closed issues when previous work or decisions may affect the request. Read likely matches rather than relying on titles.

Classify close matches by outcome:

- **Duplicate:** the same outcome. Keep one canonical issue.
- **Fragment:** a detail or step that belongs in a broader issue and has no useful delivery boundary of its own.
- **Related:** a separate outcome in the same area.
- **Dependency:** a separate outcome that must happen before or after this one.

Similar subject matter alone does not make issues duplicates.

### 3. Choose the Tracker Action

Prefer, in order:

1. Update an open issue that already covers the outcome.
2. Absorb a fragment into the canonical issue.
3. Keep independently deliverable related or dependent work separate and record the relationship when useful.
4. Create a new issue when no open issue covers the outcome.

If consolidation would materially change scope, priority, or commitments, explain the proposed change and get confirmation before editing the tracker.

### 4. Write the Issue

Keep the current structure, but use only the sections that add useful information.

#### What we need

State the desired capability, correction, or operational result. Focus on the outcome rather than the proposed implementation.

#### Current situation

Summarize the minimum verified context needed to understand the problem. Include relevant history, files, systems, or issue links only when they affect the work. State material uncertainty plainly.

#### Why this matters

Explain the practical user, business, security, or operational impact in a short paragraph.

#### What to do

Describe the scope, behaviors, constraints, and questions that must be addressed. Do not prescribe architecture, exact files, implementation steps, or new abstractions unless they are already confirmed requirements.

Encourage the assignee to study the domain documentation, current behavior, tests, and established repository patterns before choosing a solution. Do not repeat this as boilerplate when the issue already makes the discovery need clear.

#### Done when

Use a short checklist of observable results. Describe behavior and evidence, not internal design. Mention tests, documentation, access, rollout, or operational verification only when relevant to this issue.

#### Trade-offs and blind spots

Use this optional section only for meaningful constraints, open questions, dependencies, or intentionally excluded work. Do not invent speculative risks to fill the section.

### 5. Keep It Concise

- Say each fact once.
- Prefer one clear paragraph over several short paragraphs saying the same thing.
- Summarize sources instead of copying them.
- Avoid exhaustive file inventories, implementation sketches, generic test instructions, and speculative edge cases.
- Add detail in proportion to the work's complexity and risk; concise does not mean vague.

### 6. Apply Labels

Inspect the repository's labels and apply the closest labels for work type, area, and priority. Use `duplicate` only when two issues ask for the same outcome, not when work is merely related or consolidated.

## Issue Template

```markdown
## What we need

[State the outcome directly.]

## Current situation

[Summarize the relevant verified context and any material uncertainty.]

## Why this matters

[Explain the practical impact.]

## What to do

- [Describe an area, behavior, or constraint to address without designing the solution.]
- [Include necessary discovery or coordination when it is specific to this work.]

## Done when

- [ ] [Observable result]
- [ ] [Important behavior or constraint is verified]

## Trade-offs and blind spots

[Optional: meaningful uncertainty, dependency, or excluded work.]
```

## Writing Standards

Use specific, scannable titles such as `Protect the main branch from direct pushes`, not `Fix GitHub` or `Branch stuff`.

Prefer familiar words such as "manual production approval," "who can view logs," and "what image is running." Avoid phrases such as "release provenance operationalization" when direct language says the same thing.

When technical terms are necessary, use the repository's established domain language and explain unfamiliar terms briefly.

## Managing Existing Issues

- Preserve useful intent, evidence, constraints, and decisions when rewriting or consolidating.
- Summarize important decisions from comments in the body when they still govern the work.
- When closing a true duplicate, link the canonical issue in a short comment and apply `duplicate` if the repository uses it.
- When absorbing a fragment, move its unique context into the canonical issue and explain the consolidation. Do not call it a duplicate.
- Do not erase decisions unless repository evidence shows they are obsolete or wrong.

## Final Response

Briefly report which issues were created, updated, closed, or left unchanged; which likely duplicates were checked; and any unresolved choice the user needs to make. Link issue numbers when available.
