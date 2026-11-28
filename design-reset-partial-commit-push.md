# webster reset refuses before its records push when fabricSync fails after the records commit landed

`reset-partial-commit-push` · Task · bug, webster, fabric · done

## Seen

Code-Review of `webster-reset-push-sync` (finding F1, 2026-11-28) found an edge in `internal/webstercli/reset.go`.
When `fabricSync` returns an error after the records commit of the reset landed, the verb refuses before its synchronous push.
An owed push of that commit then writes no failed-push record and does not hold the run,
although the pattern documents say the reset's records push is synchronous and its owed record sets `PushOwed`.

## Fix

When `fabricSync` fails after the records commit landed, run the reset's records push (`c.pushRecords`) before refusing,
so the owed push is recorded as on the normal path; pin it with a test row beside `UnreachableRecordsOriginLeavesAnOwedRecord`.
