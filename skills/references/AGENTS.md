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
2. Use Ataski as the shared coordination board for repository task status and handoffs.
3. When using git worktrees, keep one canonical worktree in `ataski/worktrees/` (or the configured `worktreesDir`) on a dedicated canonical branch; keep that directory git-ignored; do not maintain task-local copies of the Ataski board.
4. Enforce task dependencies with `blockedBy` as a hard gate: do not claim/start a task until all `blockedBy` task IDs are `done`.
5. Prepare shared board updates in the canonical worktree first, then create one dedicated task worktree, branch, and PR per parallel task. Leverage multiple agents and sub-agents when possible.
6. Keep Ataski updated whenever shared task coordination state changes.
7. Follow the concrete task format, ID allocation, lifecycle, tracker rules, and source-code workflow gates from the `ataski` skill definition, including writing task requirements and a test plan before source edits.
8. If `ataski/config.jsonc` exists, it overrides defaults and must be followed.
9. For source-code changes, this `AGENTS.md` section is the mandate to use Ataski, and the `ataski` skill is the authoritative operational procedure (task lifecycle ordering, TDD gates, and completion checks).
10. When work finishes in a non-canonical worktree, merge against the canonical.
11. For new features or other behavior-bearing work, missing test harnesses or scaffolding status do not excuse skipping automated tests; agents must create the harness first or split out a prerequisite task.
```
