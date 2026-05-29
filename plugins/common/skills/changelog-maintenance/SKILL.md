---
description: Maintain a root CHANGELOG.md with YYYY-MM-DD headings, initializing it from git commit history when missing and appending concise dated bullets before merge.
---

# Skill: Changelog Maintenance

Use this skill when the user asks to create, backfill, repair, or update a project's root `CHANGELOG.md`. This skill is technology-agnostic and is intended for manual use before merge, release, or handoff.

## Purpose

- Keep exactly one changelog at the project root: `CHANGELOG.md`.
- Group entries under date headings in ISO format: `## YYYY-MM-DD`.
- If no changelog exists, initialize it from git history before adding current work.
- Keep summaries concise, factual, and reviewable.
- Do not update the changelog automatically unless the user explicitly invokes this skill.

---

## Required output format

Use this structure:

```markdown
# Changelog

## 2026-05-28
- Added a shared changelog-maintenance skill plugin.
- Registered the common plugin in marketplace manifests.

## 2026-05-20
- Initial project scaffolding.
```

Rules:

- The file must live at the project root.
- Use `# Changelog` as the title if creating the file.
- Date headings must be `## YYYY-MM-DD`.
- Under each date, use flat `- ` bullets only.
- Put the newest date first.
- Preserve existing manual text unless the user asked for cleanup.
- Do not add nested bullets, tables, commit hashes, or author names unless requested.

---

## Workflow

### 1. Locate and inspect

- Confirm the repository root.
- Read `CHANGELOG.md` if it exists.
- Inspect recent git history before editing so the changelog reflects actual work.

### 2. If `CHANGELOG.md` is missing or empty

Bootstrap it from git commits:

- Read git history grouped by commit date.
- Prefer commit subjects as the initial bullet text.
- Skip obvious merge-noise subjects such as generic merge commits unless they contain meaningful release information.
- Normalize bullets lightly for readability, but do not invent changes that are not supported by history.
- Create one heading per commit date and place all bullets for that date under it.

Recommended git query shape:

```sh
git log --date=short --pretty=format:'%ad%x09%s'
```

If the repository has no commits, create `CHANGELOG.md` with the title and a heading for today's date only after you have concrete changes to record.

### 3. When updating an existing changelog

- Prefer the current date unless the user requests a different date.
- Inspect the relevant source of truth for pending work:
  - committed changes not yet represented in the changelog
  - staged or unstaged diffs for the current task
  - the merge target described by the user
- Add only missing bullets.
- If today's heading already exists, append new bullets there instead of creating a duplicate heading.
- If there is no heading for today, insert a new heading at the top, below the title.

### 4. Write bullets well

Bullet guidelines:

- Summarize the outcome, not the mechanical edit.
- Keep each bullet to one line when practical.
- Prefer user-visible or reviewer-meaningful language.
- Mention filenames only when the file itself is the change of record.
- Combine tiny related edits into one bullet.
- Do not duplicate a bullet that already exists with the same meaning.

Good:

- Added a shared plugin for reusable changelog workflow instructions.
- Backfilled historical changelog entries from existing git commits.
- Clarified pre-merge workflow so changelog updates are invoked manually.

Avoid:

- Updated stuff.
- Misc fixes.
- Changed README and plugin.json.

---

## Decision rules

- If the changelog and git history disagree, preserve existing manual entries and add only what is missing.
- If commit messages are low quality, improve phrasing minimally while staying faithful to the underlying change.
- If the working tree contains uncommitted changes and the user asked for a changelog update, summarize those edits from the diff instead of pretending they are already committed.
- If there is not enough evidence to write an accurate bullet, ask the user instead of guessing.

---

## Manual invocation guidance

This skill is intended to be invoked deliberately, typically right before merge. A typical user request is:

```text
Use the changelog-maintenance skill and update the root changelog before merging.
```

When invoked, the agent should:

1. Inspect `CHANGELOG.md`.
2. Inspect git history and current diffs.
3. Update or initialize the changelog.
4. Avoid unrelated documentation rewrites.

---

## Completion checklist

- `CHANGELOG.md` exists at the project root.
- The file starts with `# Changelog`.
- Entries are grouped under `## YYYY-MM-DD` headings.
- Newest date is first.
- Historical entries were backfilled from git if the file was missing.
- The current task is represented exactly once.
- No duplicate headings for the same date.