# taskai

`taskai` is an agent skill for managing project work with the Ataski file-based task system.

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

## Install The Skill

Recommended: install per project in `.agents/skills` (not globally), so each repo keeps its own skill version.

1. From your project root, create `.agents/skills`.
2. Copy or symlink this repo as `taskai` inside `.agents/skills`.

Example:

```bash
mkdir -p .agents/skills
ln -s /absolute/path/to/this/repo .agents/skills/taskai
```

Alternative (copy instead of symlink):

```bash
mkdir -p .agents/skills
cp -R /absolute/path/to/this/repo .agents/skills/taskai
```

After install, the skill file should exist at:

```text
.agents/skills/taskai/SKILL.md
```

### Install With skills.sh

You can install this local skill with the `skills.sh` CLI:

```bash
npx skills add /absolute/path/to/this/repo -a amp
```

This installs to the project-local `.agents/skills/` path (`amp` project layout).

Use global installs only if you explicitly want cross-project sharing:

```bash
npx skills add /absolute/path/to/this/repo -a amp -g
```

## Update `AGENTS.md` To Enable Ataski

Add an entry under your `## Skills` section (or equivalent skill list):

```md
- taskai: Manage tasks with the Ataski filesystem (`ataski/todo`, `ataski/in-progress`, `ataski/done`) using incrementing IDs like `T001` and maintaining `ataski/tasks.md`. Use when creating/updating/moving project task files for agent coordination. (file: /absolute/path/to/project/.agents/skills/taskai/SKILL.md)
```

Use your real absolute path for `file:`.

## Minimal Verification

1. Confirm `AGENTS.md` includes the `taskai` entry.
2. Confirm the `file:` path points to this repo's `SKILL.md`.
3. Ask an agent to create a new task and verify it creates:
   - `ataski/todo/T00X-*.md`
   - a matching bullet in `ataski/tasks.md`
