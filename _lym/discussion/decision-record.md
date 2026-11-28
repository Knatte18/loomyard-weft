## Goal

```markdown
## Seen

Code-Review of `webster-reset-push-sync` (finding F1, 2026-11-28) found an edge in `internal/webstercli/reset.go`.
When `fabricSync` returns an error after the records commit of the reset landed, the verb refuses before its synchronous push.
An owed push of that commit then writes no failed-push record and does not hold the run,
although the pattern documents say the reset's records push is synchronous and its owed record sets `PushOwed`.

## Fix

When `fabricSync` fails after the records commit landed, run the reset's records push (`c.pushRecords`) before refusing,
so the owed push is recorded as on the normal path; pin it with a test row beside `UnreachableRecordsOriginLeavesAnOwedRecord`.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - merge-danger-omits-outward-records-push (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
