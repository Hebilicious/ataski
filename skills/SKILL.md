---
name: ataski
description: Manage project work with the Ataski filesystem board. Use whenever the user asks to use a tasklist or a todo list to coordinate work.
---

# Ataski

Use this skill to coordinate work through files in an `ataski/` directory.

## Default config.md

The following is the default configuration.

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
    "created_at": "The task creation datetime.",
    "updated_at": "The task update datetime."
  },
  "testStrategy": "Red/Green TDD",
  "tracker": "tasks.md"
}
```

### Structure

This is the default structure:

```text
ataski/
  todo/
  in-progress/
  done/
  config.md
  tasks.md
```

If it does not exist, create it before writing tasks. Note that the `config.md` file can contain instructions for customizing the default structure, so refer to it.

## Creating Tasks 

1. Allocate a task ID (ie: `T001`)
2. Creat a task name (ie: `T001-implement-typechecking`) 
3. Append that task to the tracker file (ie: `ataski/tasks.md`)
3. Create a new task file in directory 0 (ie: `ataski/todo/`)

/!\ Never reuse task ID ! /!\

## Allocate Task IDs

Task IDs must follow the `config.md` definition.

For the default format :

```json
{  
  "taskID": {
    "prefix": "T",
    "length": 3,
    "example": "T001"
  }
}
```

This means tasks ids should look like `T001`, `T002`, `T003`, ...

## Task File Naming

The user most likely won't give a name for the task, so the name must be inferred from the task content.
Always prefix the task name with the task ID.

Example : 

- `T001-short-title.md`

(Keep file names lowercase after the ID slug portion.)

## Task File Template

Every task file must be markdown with YAML frontmatter:

```md
---
id: T001
title: "Short task title"
status: todo
created_at: 2026-02-20T00:00:00Z
updated_at: 2026-02-20T00:00:00Z
---
Task content.
```

Use `status` values only from the `config.md` structure.
By default : 

- `todo`
- `in-progress`
- `done`

## Workflow 

Keep `ataski/tasks.md` as a bullet list of all tasks in ascending ID order.

Use this line format:

```md
- T001 | title-1
- T002 | title-2
```

When working on a task keep, updated

- Task frontmatter `status`
- Task directory based on `status`

## Lifecycle Rules

Use directory location and `status` together:

1. New task: created in `ataski/todo/` with `status: todo`.
2. Claimed task: move to `ataski/in-progress/` and set `status: in-progress`.
3. Completed task: move to `ataski/done/` and set `status: done`.

Always update `updated_at` when changing status or task content.

## Test Strategy

By default; each task must be written with a Red/Green TDD test Strategy. 

1. RED first: add/adjust tests that fail for the intended behavior.
2. Record RED evidence in the task content (command + failure summary).
3. GREEN second: implement the minimal code to make RED pass.
4. Record GREEN evidence in the task content (command + pass summary).
5. REFACTOR third (optional): improve code without changing behavior.
6. Update the task content immediately after each sub-step. Do not batch updates.

### TDD Enforcement

- No feature is complete unless RED and GREEN evidence exists.
- If source behavior code exists without prior RED coverage, create a backfill RED task.
- `done` status always requires tests passing

## Multi-Agent Coordination

You can add a `owner` field to the task frontmatter in the `config.md` file and in each task.

Before starting work:

1. Look for unclaimed tasks in `todo/`.
2. Move exactly one selected task from `todo/` to `in-progress/`.
3. Set `owner` to the current agent identifier if present in `config`.

Avoid editing unrelated task files unless explicitly requested.
