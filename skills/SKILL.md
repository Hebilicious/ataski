---
name: ataski
description: Manage project work with the Ataski filesystem board. Use when users ask to create, track, claim, move, or complete tasks; coordinate sub-agents with git worktrees; enforce task dependencies with `blockedBy`; or maintain `ataski/tasks.md`.
---

# Ataski

Use this skill to coordinate work through files in an `ataski/` directory.

## Canonical Board Rule

When using git worktrees, keep one canonical `ataski/` board in the main worktree.
Do not create independent copies of the `ataski/` board per worktree.
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

## Test Strategy

By default, each source-code task follows Red/Green TDD:

1. RED first: add or adjust failing tests.
2. Record RED evidence in the task file (command + failure summary).
3. GREEN second: implement minimal changes to pass.
4. Record GREEN evidence in the task file (command + pass summary).
5. REFACTOR optional: improve code without behavior changes.

Do not mark task `done` without passing tests for source-level work.

## Multi-Agent Coordination

Core rules:

1. Use the main worktree as the canonical location for shared `ataski/` updates.
2. Keep one task per worktree branch (example: `task/T001-short-title`).
3. Keep one PR per task branch.

Follow this sequence for each task:

1. In main worktree, pick one `todo` task with all `blockedBy` tasks in `done`.
2. Claim the task on the canonical board before coding:
   - move file to `ataski/in-progress/`
   - set `status: in-progress`
   - set `owner` when used
   - update `updated_at`
   - update matching line in `ataski/tasks.md`
3. Create branch/worktree for only that task (`task/<ID>-<slug>`).
4. Assign exactly one agent or sub-agent to that worktree/task.
5. The assigned agent or sub-agent implements only the claimed task scope and opens one PR.
6. After merge, in main worktree:
   - move file to `ataski/done/`
   - set `status: done`
   - update `updated_at`
   - update `ataski/tasks.md`
7. Re-evaluate blocked tasks and claim newly unblocked work.

Coordination constraints:

- Do not claim blocked tasks.
- Do not claim more than one task per agent or sub-agent at a time.
- Do not edit unrelated task files unless explicitly requested.
- If two agents race for the same task, the first canonical board update wins; other agents must re-pick from `todo`.
