# ataski

`ataski` is an agent skill for managing project work with a file-based task system.

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

The `ataski` skill is opinionated:

- Use Red/Green TDD: start with a failing test, then implement until it passes.
- Use worktrees and multiple agents/sub-agents by default. For large task sets, specify the number of sub-agents in the prompt to control parallelism.

## Install The Skill

Install with `skills.sh`:

```bash
npx skills add Hebilicious/ataski
```

This installs `ataski` to the project-local `.agents/skills/ataski/` path.

## Update `AGENTS.md` To Enable Ataski

Update your agent instruction file using the reference provided by the `ataski` skill:

- `skills/references/AGENTS.md`

## Minimal Verification

1. Confirm your agent instruction file is updated from `skills/references/AGENTS.md`.
2. Confirm the `file:` path points to `.agents/skills/ataski/SKILL.md`.
3. Ask an agent to create one temporary task and verify it creates:
   - `ataski/todo/T00X-*.md`
   - a matching bullet in `ataski/tasks.md`
4. Delete the temporary task file immediately after verification and remove its matching line from `ataski/tasks.md`.

## Customization

`ataski` is intentionally simple and meant to be customized.
Modify the skill definition in `.agents/skills/ataski/SKILL.md` and the added instructions in your `AGENTS.md` file to suit your needs.

## Claude Code

For Claude Code, ensure `.claude/skills/ataski` points to the installed `ataski` skill directory.

And add the following to your `CLAUDE.md` file:

```md
@AGENTS.md
```
