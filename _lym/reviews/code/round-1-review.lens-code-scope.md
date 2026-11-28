```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to the board entry's goal points.

**Decisions** (the record holds a decision record alone and no `Added after Discussion` entries):
- G1: when `fabricSync` fails after the records commit landed, run the reset's records push (`c.pushRecords`) before refusing, so an owed push is recorded as it is on the normal path (`_lym/discussion/decision-record.md:13-14`).
- G2: pin G1 with a test row beside `UnreachableRecordsOriginLeavesAnOwedRecord` (`_lym/discussion/decision-record.md:14`).

**Per change:**
- C1, the failure branch in `internal/webstercli/reset.go:194-206`. When `committed` is true, it calls the push before `fail(msg)`, and it returns `push_error` in the refusal envelope when the push fails. This traces to G1. `push_error` is how the normal path reports a failed push (`reset.go:216-217`), so carrying it matches "recorded as on the normal path".
- C2, the `pushResetRecords` extraction in `reset.go:247-260` and the success-path switch in `reset.go:213-218`. C1 forces this: both paths now share the skip conditions, the call and the warning log. The success path's behavior is kept: the same skip guard, a hand-over still fails, and the warning is still logged for any other failure, now inside the method.
- C3, the docs: the package header in `reset.go:6-7`, the help text in `reset.go:65-67`, and the new line in `internal/webstercli/doc.go:35`. C1 forces these, since the existing statements that the push runs before the verb returns would otherwise miss the refusal case.
- C4, the test in `internal/webstercli/reset_test.go`: the `syncFailsAfterCommit` field, the row `SyncFailingAfterTheCommitLeavesAnOwedRecord`, the setup that squats the corrindex path, the branch asserting the refusal, and the doc-comment line at 168. This traces to G2, with the row placed directly beside `UnreachableRecordsOriginLeavesAnOwedRecord` (lines 188-189).
- C5, commit 15d53d1 ("break the reset comments at sentence boundaries"). It only re-wraps the comments C3 added, so C3 forces it.

**Per decision:**
- G1 is built by C1, and C2, C3 and C5 are forced by it.
- G2 is built by C4.

**Findings:** none. Every change traces to G1 or G2 or is forced by a traced change, and both goal points are built.

Tool calls: none.
