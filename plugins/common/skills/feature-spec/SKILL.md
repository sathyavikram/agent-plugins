---
description: Create the next feature spec from specs/roadmap.md by making a branch, asking one grouped feature-spec question set, and writing dated plan, requirements, and validation files.
---

# Skill: Feature Spec

Use this skill when the user wants to stop retyping the same feature-spec prompt and create the next implementation spec in a consistent way. This skill is technology-agnostic and fits any project that keeps a `specs/` directory with constitution-style guidance.

## Purpose

- Read the next planned phase from `specs/roadmap.md`.
- Create a working branch for that feature before writing the new spec.
- Gather missing spec inputs with exactly one grouped Ask User Question step.
- Create a dated spec directory: `specs/YYYY-MM-DD-feature-name/`.
- Write three files in that directory: `plan.md`, `requirements.md`, and `validation.md`.
- Keep the new spec aligned with `specs/mission.md` and `specs/tech-stack.md`.

---

## Required prerequisites

Before writing any spec files:

- Confirm `specs/roadmap.md` exists.
- Read `specs/mission.md` if present.
- Read `specs/tech-stack.md` if present.
- Identify the next incomplete roadmap phase from `specs/roadmap.md`.

If `specs/roadmap.md` does not exist, stop and ask the user to create the project constitution first.

---

## Mandatory interaction rule

You must use the Ask User Question tool before writing to disk.

The questions must be grouped around these three outputs:

1. `plan.md`
2. `requirements.md`
3. `validation.md`

Do not create the new spec directory or any files until the user has answered that grouped question set.

The grouped questions should collect:

- `plan.md`: task groups, implementation order, dependencies, and any branch or rollout constraints
- `requirements.md`: scope, decisions, open constraints, non-goals, context, naming, and any exact acceptance boundaries
- `validation.md`: tests, checks, demos, review steps, and merge criteria

If enough information is already explicit in the roadmap phase, ask only for the missing pieces, but still keep the questions grouped on those three files.

---

## Workflow

### 1. Read the spec context

- Read `specs/roadmap.md`.
- Read `specs/mission.md`.
- Read `specs/tech-stack.md`.
- Determine the next incomplete roadmap phase.

### 2. Create the branch

Before writing files:

- Derive a short kebab-case branch name from the next roadmap phase.
- Create and switch to that branch.
- Prefer a date-prefixed or feature-prefixed branch name only if the repository already follows that pattern.

If branch creation is not possible in the current environment, tell the user and continue only if they want the spec files written without the branch step.

### 3. Ask the grouped feature-spec questions

Use one Ask User Question call that covers:

- plan
- requirements
- validation

Keep the questions concise and practical. Do not ask broad brainstorming questions when the roadmap phase already narrows the work.

### 4. Create the dated spec folder

Create:

```text
specs/YYYY-MM-DD-feature-name/
```

Rules:

- Use today’s date unless the user specifies a different date.
- Convert the feature name to lowercase kebab-case.
- Use a short, stable name derived from the roadmap phase and user answers.

### 5. Write the three files

Create exactly these files:

- `plan.md`
- `requirements.md`
- `validation.md`

#### `plan.md`

Requirements:

- Use numbered task groups.
- Order tasks from enabling work to implementation to verification.
- Keep groups small enough to implement and review incrementally.
- Reflect the roadmap phase and any dependencies from the user.

Suggested structure:

```markdown
# Plan

## Task Group 1
1. ...
2. ...

## Task Group 2
1. ...
2. ...
```

#### `requirements.md`

Requirements:

- Capture scope, decisions, assumptions, and context.
- Include non-goals when they help prevent scope drift.
- Reflect constraints from `mission.md`, `tech-stack.md`, and the user’s answers.
- Be concrete enough that implementation can proceed without re-deriving the intent.

Suggested structure:

```markdown
# Requirements

## Scope

## Decisions

## Constraints

## Non-goals

## Context
```

#### `validation.md`

Requirements:

- Define how to prove the work is ready to merge.
- Include executable checks where practical.
- Include any manual review or demo steps that matter.
- Distinguish required checks from optional follow-up checks when needed.

Suggested structure:

```markdown
# Validation

## Required Checks

## Manual Review

## Merge Criteria
```

---

## Writing rules

- Use the roadmap phase as the anchor for naming and scope.
- Prefer precise, reviewable language over aspirational wording.
- Do not write implementation code as part of this skill.
- Do not modify unrelated spec folders.
- Do not mark the roadmap phase complete during spec creation.
- Do not skip the Ask User Question step.

---

## Example invocation

```text
Use the feature-spec skill. Find the next phase on specs/roadmap.md, make a branch, ask me the grouped feature-spec questions, and create the dated spec folder.
```

---

## Completion checklist

- `specs/roadmap.md` was read.
- `specs/mission.md` and `specs/tech-stack.md` were consulted if present.
- The next incomplete roadmap phase was identified.
- A branch was created or the blocker was surfaced to the user.
- One grouped Ask User Question step was used before writing files.
- `specs/YYYY-MM-DD-feature-name/` was created.
- `plan.md`, `requirements.md`, and `validation.md` were created.
- `plan.md` uses numbered task groups.
- The spec matches the roadmap phase and user answers.