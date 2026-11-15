## Goal

```markdown
Imported from [issue #121](https://github.com/Knatte18/loomyard/issues/121).

## Seen

A raddle filing left five tasks with `discussed: true` and a stored decision record each.
Before they started, the orch added sections to the board bodies of four of them through `lym board upsert --body-file` (a gate test budget and a step benchmark).
The upsert succeeded with no warning.
The first of those tasks to run planned from its stored record alone: no card carried the body's new sections, the plan review did not flag them, and the PR missed the benchmark and broke the test budget.
The gap surfaced only in the parent's PR review.

## Workaround

Before such a task starts, `lym loom decision add` refuses (no worktree yet), so the orch filed `Decision for <slug>: ...` notes and plans to record the same content with `lym loom decision add` once each run exists.

## Expected

A body edit on a discussed entry that the stored record does not carry should either reach the run (Plan-Write and the plan review read the body beside the record) or be refused or warned about at upsert, naming `lym loom decision add` or another way to amend the stored record before the run starts.
Related: #116, #117.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - body-edit-warning-guards-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
