---
name: AGENTS.md Reference
description: You should update the AGENTS.md file to include Ataski.
---

Use this section in `AGENTS.md` to enforce Ataski usage:

```md
## Non-Negotiable Workflow

Scope: this workflow is mandatory for source code changes (runtime, business logic, and tests).  
For chore/tooling/config/docs-only changes, this workflow is optional unless explicitly requested.

1. To keep track and organize your work, you must use the `ataski` skill.
2. Treat Ataski as the only task-tracking source of truth for the repository.
3. When using git worktrees, keep one canonical `ataski/` board in the main worktree; do not maintain per-worktree task boards.
4. Enforce task dependencies with `blockedBy` as a hard gate: do not claim/start a task until all `blockedBy` task IDs are `done`.
5. For each task, use a dedicated worktree, branch, and PR. Leverage multiple agents and sub-agents when possible.
6. Do not use or create parallel trackers (for example: `TASKLIST.md`, ad-hoc TODO files, or duplicate status boards) unless explicitly requested by the user.
7. Follow the concrete task format, ID allocation, lifecycle, and tracker rules from the `ataski` skill definition.
8. If `ataski/config.md` exists, it overrides defaults and must be followed.
```
