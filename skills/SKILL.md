---
name: ataski
description: Manage project work with the Ataski filesystem board. Use when users ask to create, track, claim, move, or complete tasks; coordinate sub-agents with git worktrees; enforce task dependencies with `blockedBy`; or maintain `ataski/tasks.md`.
---

# Ataski

Use this skill to coordinate work through files in an `ataski/` directory.

## Canonical Board Rule

When using git worktrees, keep one canonical worktree in the configured worktree directory (`worktreesDir`, default: `ataski/worktrees/`).
The canonical worktree MUST use a dedicated canonical branch (example: `ataski/canonical`).
Keep the shared `ataski/` board only in that canonical worktree.
Do not create independent copies of the `ataski/` board in task worktrees.
Prepare or update the board in the canonical worktree first, then create isolated task worktrees for parallel execution.
The configured worktree directory MUST be git-ignored.

## Default `config.jsonc`

If `ataski/config.jsonc` exists, it overrides defaults.
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
  "tracker": "tasks.md",
  "worktreesDir": "ataski/worktrees/"
}
```

The same default file contents are mirrored in `skills/references/config.jsonc`.

## Structure

Default structure:

```text
ataski/
  todo/
  in-progress/
  done/
  worktrees/
  config.jsonc
  tasks.md
```

If it does not exist, create it before writing tasks.

The default `ataski/worktrees/` directory is reserved for git worktrees and MUST be git-ignored.

## Worktree Configuration

Use `worktreesDir` to control where git worktrees are created.

- Default: `ataski/worktrees/`
- The canonical worktree lives inside `worktreesDir`.
- Each parallel task gets its own separate worktree inside `worktreesDir`.
- The canonical branch should use an explicit shared name such as `ataski/canonical`.
- Task branches should stay task-scoped (example: `task/T001-short-title`).

## Create Tasks

1. Allocate the next task ID (example: `T001`).
2. Infer a short slugged title (example: `T001-implement-typechecking.md`).
3. Create a new task file in directory `0` (default: `ataski/todo/`).
4. Append a matching entry to `ataski/tasks.md`.

Never reuse task IDs.

## Allocate Task IDs

Task IDs must follow `config.jsonc`.
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
## Requirements

- Describe the concrete behavior, acceptance criteria, and constraints.

## Test Plan

- Describe the intended RED path before implementation (tests, harness updates, or prerequisite test setup).

## RED Evidence

## GREEN Evidence
```

Use `status` values from configured structure.
Default statuses:

- `todo`
- `in-progress`
- `done`

For source-code tasks, the task body MUST be specific enough to execute without guessing. A task file is not ready for implementation if `Requirements` or `Test Plan` is missing or vague.

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

## Source-Code Scope and Hard Gates

The rules in this section apply to **source-code changes** and are hard-gated.

Source-code changes include changes that affect runtime behavior, business logic, test behavior, or execution paths, including examples such as:

- route rewiring
- deleting runtime code paths
- service rewrites
- data-path migrations
- changing tests that validate behavior

Chore-only changes are not source-code changes unless they alter behavior, including examples such as:

- `README` edits
- `.gitignore` edits
- lockfile updates (unless they intentionally change runtime behavior)
- formatting-only or comment-only edits

For source-code changes, the agent MUST follow this ordered workflow and MUST NOT skip or reorder the gates:

1. Create task in `ataski/todo/` (`status: todo`).
2. Write `Requirements` and `Test Plan` in the task file.
3. Move task to `ataski/in-progress/` (`status: in-progress`).
4. Record RED evidence in the task file, or (if TDD is truly not applicable) document a `TDD Exception` before source edits.
5. Implement code changes.
6. Record GREEN evidence in the task file.
7. Move task to `ataski/done/` (`status: done`).
8. Update `ataski/tasks.md` to reflect the final completed state and path.

Apply the normal tracker/frontmatter updates when status changes (create/claim/complete). Step 8 is the required final tracker sync/check.

Pre-edit coordination rule (source-code changes):

- Before editing any source file, the task MUST already exist, MUST already contain concrete `Requirements` and `Test Plan` sections, and MUST already be in `ataski/in-progress/`.

No retroactive tracking (source-code changes):

- Agents MUST NOT create, claim, or move a task to `done` after the source edits are already complete.
- Agents MUST NOT treat Ataski updates as retroactive paperwork for already-finished source-code work.

## Test Strategy

By default, each source-code task follows Red/Green TDD:

1. RED first: add or adjust failing tests.
2. Record RED evidence in the task file (command + failure summary).
3. GREEN second: implement minimal changes to pass.
4. Record GREEN evidence in the task file (command + pass summary).
5. REFACTOR optional: improve code without behavior changes.

TDD gate requirements for source-code changes:

- RED evidence MUST be recorded before implementation edits.
- If TDD is not applicable, add a `TDD Exception` section in the task file **before source edits**. This section MUST explain why TDD is not applicable and what validation will be used instead.
- A `TDD Exception` replaces the RED requirement only. It does not waive GREEN validation.
- GREEN evidence is always required before moving the task to `done`.
- For new features, new packages, service rewrites, route rewiring, and other behavior-bearing work, automated tests are mandatory. Build, lint, typecheck, and manual smoke checks alone do NOT satisfy this requirement.
- Missing test coverage, missing test harnesses, greenfield package setup, or scaffolding work are NOT valid reasons to skip TDD. The agent MUST first add or extend a test harness, or create a prerequisite task to do so, and then continue with RED/GREEN.
- A full feature or public package MUST NOT use a blanket `TDD Exception` to avoid writing tests for its user-visible behavior.
- If only part of a task cannot be tested first, the exception MUST be narrowly scoped to that part, and the agent MUST still write tests for the remaining behavior or split the work into smaller tasks.

Do not mark task `done` without passing tests (or the documented exception validation) for source-level work.

## Completion Compliance Checklist

Before completing a source-code task, verify all items below:

- task was created before source edits
- task requirements and test plan were written before source edits
- task was moved to `in-progress` before source edits
- RED and GREEN evidence were recorded, or a pre-edit `TDD Exception` plus GREEN evidence was recorded
- task was moved to `done` only after implementation and validation were complete

## AGENTS.md Interaction

Keep and use the `AGENTS.md` guidance (see `skills/references/AGENTS.md`) to mandate Ataski at the project level.

For source-code changes:

- `AGENTS.md` provides the project mandate/policy to use Ataski.
- The `ataski` skill provides the exact operational procedure, lifecycle ordering, and hard gates.
- If `AGENTS.md` says to use Ataski, then the Ataski lifecycle and TDD/compliance gates in this skill are mandatory.

## Multi-Agent Coordination

Core rules:

1. Use one canonical worktree inside `worktreesDir` as the shared location for `ataski/` updates.
2. The canonical worktree MUST use a dedicated canonical branch (example: `ataski/canonical`).
3. Keep one task worktree per task branch under `worktreesDir` (example branch: `task/T001-short-title`).
4. Keep one PR per task branch.

Follow this sequence for each task:

1. In the canonical worktree, pick one `todo` task with all `blockedBy` tasks in `done`.
2. In the canonical worktree, claim the task on the shared board before coding:
   - move file to `ataski/in-progress/`
   - set `status: in-progress`
   - set `owner` when used
   - update `updated_at`
   - update matching line in `ataski/tasks.md`
3. From the canonical branch, create a new task branch/worktree for only that task under `worktreesDir` (`task/<ID>-<slug>`).
4. Assign exactly one agent or sub-agent to that worktree/task.
5. The assigned agent or sub-agent implements only the claimed task scope and opens one PR.
6. When work is finished in a non-canonical worktree, suggest review by comparing the task branch against the canonical branch before merge.
7. After merge, return to the canonical worktree:
   - move file to `ataski/done/`
   - set `status: done`
   - update `updated_at`
   - update `ataski/tasks.md`
8. Re-evaluate blocked tasks and claim newly unblocked work.

Coordination constraints:

- Do not claim blocked tasks.
- Do not claim more than one task per agent or sub-agent at a time.
- Do not edit unrelated task files unless explicitly requested.
- If two agents race for the same task, the first canonical board update wins; other agents must re-pick from `todo`.

Git/worktree policy prerequisite:

- If project policy (for example `AGENTS.md`) requires git and/or worktrees for task execution and they are unavailable, the agent MUST stop before source edits and ask the user to initialize git or enable the required worktree workflow.
