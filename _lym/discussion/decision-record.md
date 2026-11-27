## Goal

```markdown
Seen on weaver-bench-b2 (2026-11-27): Webster Master's first prompt named its friction note `_lym/loom/friction/webster-master.md`, and Master wrote it.
After `lym loom resume` the new Master prompt named the note `webster-master-2.md`.
The run-exit audit then flagged the earlier, legitimate note as a parent-write and blocked the Webster row, although the audit itself reported the path as cleared, unchanged since Master spawned.
The orch had to run `lym webster accept-audit` by hand.

Idea: the audit treats every friction note path a Master of this run was given as Master's own, not only the latest prompt's.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
