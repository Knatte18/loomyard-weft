## Fixer-report, round 1

### Changes

- F1 (fixed as proposed): `TestGateTest_Scopes` gained a `--card` watch subtest in `internal/gatecli/scopes_integration_test.go`.
  It asserts `-skip ^(TestWatched)$` with a parent branch, no skip once `watched/x_linux.go` changes, and no skip on the prime, which has no parent branch.
  The subtest removes the prime's plan and green record on cleanup, so the later prime row still sees a clean prime.
- F2 (fixed as proposed): `TestScanGuards_DropsWatchedGuardsNoReadChangeConcerns` covers `scanGuards` and `moduleRelative` for the root and a nested module dir.
- F3 (fixed as proposed, second option): `pattern/PATTERN-test-speed.md` "Time is injectable" now names the `internal/gitexec` seam exception.
- F4 (fixed as proposed): `ArmDefaultDeadlineForTest` in `export_test.go` and a `TestRun_RemoteBounds` row assert the production deadline ends its context with `context.DeadlineExceeded`.
- F5 (fixed as proposed): the long command is `sleep 30 & echo up; wait`.
  On Windows it is `ping -n 30 127.0.0.1`, whose banner prints only once ping runs.
- F6 (fixed differently from the proposal's placement): the guard bullet itself now carries the `//lym:watch` exception, in place of a separate bullet that left the first one stale.
- F7 (fixed differently): the proposal said to drop `%v` of err from the GuardMisplaced case.
  That left the refusal without the marker's identity, and the watch-malformed twin still doubled its way forward.
  The guard-misplaced scan error in `internal/impactset/guards.go` now carries its own way forward, and one refusal case for `GuardMisplaced` and `WatchMalformed` in `guardScanRefusal` prints the scan error once.
  The pinned text in `test_integration_test.go` (the watch-malformed row) changed to the scan error's clause.

Gate: `lym gate test --tier 1` over `internal/gatecli`, `internal/impactset`, `internal/gitexec` passed with 0 failures; the F4 and F5 packages passed in the run before.

### Self-check

- fix-code-correctness: returned no findings.
- fix-consistency, MEDIUM, WatchMalformed refusal still doubled its way forward: fixed (merged case above).
- fix-consistency, LOW, the guard bullet still said "whatever the impacted set": fixed (exception folded into the bullet).
- fix-consistency, LOW, `export_test.go` header silent on the deadline hook: fixed.
- fix-affected-set, MEDIUM, the same stale bullet (duplicate of the fix-consistency finding): fixed.
- fix-affected-set, LOW, `internal/impactset/doc.go:42` does not mention the watched-guard drop: disputed, premise false.
- fix-proposal, LOW, F7 refusal loses the marker's identity and over-constrains a watch marker: fixed by the F7 change above.

### Disputed

- fix-affected-set LOW, `internal/impactset/doc.go`: premise false.
  Lines 64-69 state that `Guards` leaves out the watched tests the diff leaves alone.

### Deferred

Nothing deferred.
