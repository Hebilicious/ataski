---
name: ataski
description: Manage project work with the Ataski filesystem board. Use when users ask to create, track, claim, move, or complete tasks; coordinate sub-agents with git worktrees; enforce task dependencies with `blockedBy`; or maintain `ataski/tasks.md`.
---

# Ataski

Use this skill to coordinate work through files in an `ataski/` directory.

## Canonical Board Rule

When using git worktrees, keep one canonical `ataski/` board in the main worktree.
Do not create independent task boards per worktree.
Sub-agents claim tasks from the canonical board, then implement in isolated worktrees.

## Default `config.md`

If `ataski/config.md` exists, it overrides defaults.
Default configuration:

```json
{
  "structure": {
    "0": "todo/",
    "1": "in-progress/",
    "2": "done/"
  },
  "taskID": {
    "prefix": "T",
    "length": 3,
    "example": "T001"
  },
  "taskFrontMatter": {
    "id": "The task ID",
    "title": "The task title",
    "status": "The task status",
    "blockedBy": "Array of task IDs that must be done before this task can start",
    "owner": "Optional current owner/agent",
    "created_at": "The task creation datetime",
    "updated_at": "The task update datetime"
  },
  "testStrategy": "Red/Green TDD",
  "tracker": "tasks.md"
}
```

## Structure

Default structure:

```text
ataski/
  todo/
  in-progress/
  done/
  config.md
  tasks.md
```

If it does not exist, create it before writing tasks.

## Create Tasks

1. Allocate the next task ID (example: `T001`).
2. Infer a short slugged title (example: `T001-implement-typechecking.md`).
3. Create a new task file in directory `0` (default: `ataski/todo/`).
4. Append a matching entry to `ataski/tasks.md`.

Never reuse task IDs.

## Allocate Task IDs

Task IDs must follow `config.md`.
For default config, IDs are `T001`, `T002`, `T003`, ...

Find the next ID by reading `ataski/tasks.md`, taking the highest existing ID, and incrementing it.
If no tasks exist, start at `T001`.

## Task File Naming

Always prefix the file name with the task ID.

Example:

- `T001-short-title.md`

Keep file names lowercase after the ID slug.

## Task File Template

Every task file must be markdown with YAML frontmatter:

```md
---
id: T001
title: "Short task title"
status: todo
blockedBy: []
owner: unassigned
created_at: 2026-02-21T00:00:00Z
updated_at: 2026-02-21T00:00:00Z
---
Task content.
```

Use `status` values from configured structure.
Default statuses:

- `todo`
- `in-progress`
- `done`

## `blockedBy` Rules

`blockedBy` controls dependencies between tasks.

1. Use `blockedBy` as an array of task IDs (example: `blockedBy: [T001, T004]`).
2. Default to an empty array for independent tasks.
3. Reference only IDs that exist in `ataski/tasks.md`.
4. Never include the task's own ID.
5. Reject dependency cycles (direct or transitive) when creating or editing tasks.
6. Treat dependencies as a hard gate: do not claim/start a task until every ID in `blockedBy` is `done`.

## Tracker Workflow

Keep `ataski/tasks.md` as a bullet list in ascending ID order.

Use this line format:

```md
- T001 | todo | title-1 | todo/T001-title-1.md
```

When task state changes, update both:

- task frontmatter (`status`, `updated_at`, optional `owner`)
- matching `ataski/tasks.md` line

## Lifecycle Rules

Use directory location and `status` together:

1. New task: create in `ataski/todo/` with `status: todo`.
2. Claimed task: move to `ataski/in-progress/` with `status: in-progress`.
3. Completed task: move to `ataski/done/` with `status: done`.

Always update `updated_at` when status or task content changes.

## Worktree + Sub-Agent Workflow

1. In the main worktree, create tasks and define dependencies with `blockedBy`.
2. Select only tasks that are not blocked.
3. Claim the task on the canonical board (`todo -> in-progress`) before coding.
4. Create one branch/worktree per task (example branch: `task/T001-short-title`).
5. Assign exactly one sub-agent per claimed task/worktree.
6. Sub-agent implements only that task and opens one PR for that branch.
7. After merge, update canonical board in main worktree (`in-progress -> done`), then re-check blocked tasks.

## Test Strategy

By default, each source-code task follows Red/Green TDD:

1. RED first: add or adjust failing tests.
2. Record RED evidence in the task file (command + failure summary).
3. GREEN second: implement minimal changes to pass.
4. Record GREEN evidence in the task file (command + pass summary).
5. REFACTOR optional: improve code without behavior changes.

Do not mark task `done` without passing tests for source-level work.

## Multi-Agent Coordination

Before starting work:

1. Read canonical board from main worktree.
2. Pick one unblocked `todo` task.
3. Move exactly one task to `in-progress`.
4. Set `owner` if used in config.

Avoid editing unrelated task files unless explicitly requested.
