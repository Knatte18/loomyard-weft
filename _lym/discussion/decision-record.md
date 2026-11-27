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

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - merge-danger-two-way-despite-state-key-rename (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
