# Lens review

- criteria-coverage, MEDIUM, D1 key order untested: fixed; card 1's set-status row gains `saveOrder`, pinning kept keys after the known fields in sorted order.
- criteria-coverage, LOW, D7 budget length unpinned: fixed; card 3's budget row asserts exactly `int(removeRetryBudget/removeRetryDelay) + 1` attempts on the first directory.
- forced-edits, LOW, run-lock comments go stale: fixed; card 1 lists `checkRunLocks` and `scopeChanged` and says their comments name `Unknown` as never compared.
- forced-edits, LOW, `Task.MarshalJSON` promoted onto `TaskWithLayer` (`internal/boardengine/layer.go:152`): fixed; card 1 lists `TaskWithLayer` and has its comment say it is never JSON-encoded.
- forced-edits, LOW, `ApplyPatch` doc says "existing → map" (`internal/boardengine/task.go:161`): fixed; card 1's Intent updates that comment.
- row-reachability, MEDIUM, card 2 rows cannot see exact digits through `assertResult`'s pre-decoded map (`internal/boardcli/cli_test.go`, `stepLookupContract` runner): fixed; the table gains `assertStdout` taking raw stdout.
- row-reachability, LOW, a `TestLoad` row without `check` panics: fixed; the runner skips a nil `check`.
- row-reachability, LOW, merge and remove rows checked `zz_b` only where it trivially vanishes: fixed; they assert `zz_b` absent on `a` and at the top level.
- row-reachability, LOW, card 3 claimed a row fails on today's code that has no seam: fixed; the sentence is cut.
- test-economy: no findings.
