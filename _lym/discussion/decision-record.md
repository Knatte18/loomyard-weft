## Goal

```markdown
Imported from [issue #188](https://github.com/Knatte18/loomyard/issues/188).

Seen: lym board list --text showed a task that had landed in a beam project under P0, the root project, instead of under its beam.

Workaround: none; the entry was left until prune.

Expected: a done task stays listed under its own project until it is pruned.
```

## Scope

## Decisions

### Added after Discussion (arbiter, 2026-11-23): The task fixes the board viewer; the list --text listing stays as it is

Decision: The task covers the board viewer (board.html and the board-data.js projects list from viewProjects in internal/boardengine/view.go) only. The lym board list --text listing in internal/boardcli/text.go and its P0 row placement by withRootProjectRow are left unchanged, and no review round raises their absence as a finding again (F3 of code review round 1, list-text-unchanged-goal-narrowed, is settled by this decision).

Rationale: The entry's title names the board view. The --text listing is flat: it shows no project for any row, and RenderOrder puts every done entry after the P0 row whatever its project, so no row there stands under a project except by order. Only the viewer places an entry under its project, and only there did a done beam project's member fall back to P0. Making --text group entries under their projects would be a new listing design that the board entry does not specify, which would widen the task.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
