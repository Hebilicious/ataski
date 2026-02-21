# ataski

`ataski` is an agent skill for managing project work with the Ataski file-based task system.
GitHub repository: `Hebilicious/ataski`.

## What Ataski Is

Ataski uses a simple directory structure:

```text
ataski/
  todo/
  in-progress/
  done/
  tasks.md
```

Each change gets a new markdown task file with frontmatter.
Task IDs must increment and start with a letter plus number (for example: `T001`, `T002`).
`ataski/tasks.md` is the canonical bullet list index for all tasks.
Task dependencies are declared in frontmatter using `blockedBy`.

Example:

```yaml
---
id: T010
title: "Integrate OAuth callback validation"
status: todo
blockedBy: [T008]
created_at: 2026-02-21T00:00:00Z
updated_at: 2026-02-21T00:00:00Z
---
```

`blockedBy` is a hard gate: if blockers are not `done`, the task stays in `todo`.
Avoid circular dependencies in `blockedBy`.

## Worktrees + Sub-Agents (Recommended)

Use a single canonical task board in the main worktree, then execute each task in an isolated worktree.

1. In main worktree, create tasks in `ataski/todo/` and set dependencies with `blockedBy`.
2. Select only unblocked tasks (all `blockedBy` IDs must be `done`).
3. Claim task by moving it to `ataski/in-progress/` before coding.
4. Spawn one worktree branch per task (for example `task/T014-add-oauth-callback-validation`).
5. Assign one sub-agent to that worktree and task.
6. Merge one PR per task branch.
7. In main worktree, move merged task to `ataski/done/`, update `tasks.md`, then re-check newly unblocked tasks.

## Install The Skill

Install with `skills.sh`:

```bash
npx skills add Hebilicious/ataski
```

This installs `ataski` to the project-local `.agents/skills/ataski/` path.

## Update `AGENT.md` / `AGENTS.md` To Enable Ataski

Update your agent instruction file using the reference provided by this skill:

- `skills/references/AGENTS.md`

## Minimal Verification

1. Confirm your agent instruction file is updated from `skills/references/AGENTS.md`.
2. Confirm the `file:` path points to `.agents/skills/ataski/SKILL.md`.
3. Ask an agent to create one temporary task and verify it creates:
   - `ataski/todo/T00X-*.md`
   - a matching bullet in `ataski/tasks.md`
4. Delete the temporary task file immediately after verification and remove its matching line from `ataski/tasks.md`.
