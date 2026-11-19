```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test row that pins it.

**Per behaviour, the test that pins it:**
1. `proc.BootTime` on Linux reads `btime`: pinned by `TestBootTime` (`internal/proc/proc_linux_test.go:66`).
2. `proc.BootTime` on Linux fails on a missing or unparsable `btime` line: no test. The path is the fixed constant `/proc/stat` with no seam, so a test cannot reasonably reach it. No finding.
3. `proc.BootTime` on Windows: no test. `internal/proc/proc_windows_test.go` exists but has no `BootTime` case. See F2.
4. A refusal to step leaves the "ends without an envelope" mark in the loop log: pinned by `TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog`, row "a dead loop's pid file refuses the step".
5. A lost loop lock leaves its mark, including the reason's `: <err>` suffix branch: no row asserts it. `TestUntilStop_LoserWaitsOnWinner` reaches this path (reach report) but asserts only the winner's envelope and the count of started children (`loopwait_test.go:1074-1078`). The shared `endWithoutEnvelope` mark is pinned by row 1, so only this path's reason text is unpinned. See F3.
6. The directory-creation and pid-write failure marks: no row. They share the helper pinned by behaviour 4 and differ only in the literal reason text. No separate finding.
7. `endWithoutEnvelope` returns exit code 1, which both its doc and `runLoop`'s doc state: no test. The mutation survivor at `loop.go:545` confirms it. See F1.
8. A failed envelope write leaves a mark: pinned by `TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog`, row "an envelope path the loop cannot write".
9. The pid file records `StartedAt` from `LoopSpec.Now`, and the in-flight child update keeps it: pinned by the same row. The loop steps once, the child-start rewrite runs, then the envelope write fails, the pid file stays, and the row asserts `StartedAt == loopStart`.
10. `deadLoopStop` passes `rec.StartedAt` to `unmarkedExitCause`, which names a boot after the start, a boot before it, or both when the boot time is unknown: pinned by `TestUntilStop_DeadLoopStopsLoopExited` rows `UnmarkedExit_BootAfterTheStart_NamesAStopOfTheMachine`, `UnmarkedExit_BootBeforeTheStart_NamesAKill` and `UnmarkedExit_StartKnownBootUnknown_NamesBoth`.
11. An older pid file without `started_at` falls back to the old wording: pinned by the existing row with a zero `startedAt`. `omitzero` drops the key, so that file has the older shape.
12. `loopExitedEarlyStop` and the integration callers pass a zero start: pinned by the existing `TestLoopIntegration_*` regex assertions and the early-exit rows.
13. `loopAgreesWithState` treats held beside busy on background work as agreement: pinned by the new `TestWait_LogsSessionStateBesideItsClassification` row. `wantChanges` holds `"busy/background held"`, and `wantDisagreements` defaults to 0, which `waitshadow_test.go:268` asserts.

**Findings:**

- F1. LOW, design. Location: `internal/shedverbs/loop_test.go` `TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog`. The test never asserts that a loop ending without an envelope exits with code 1, though the docs of `endWithoutEnvelope` and `runLoop` state that code. Evidence: the mutation survivor at `internal/shedverbs/loop.go:545`, and the test (diff lines 186-206) discards the result of `clihelp.Execute`. Fix: assert the exit code in each row.
- F2. LOW, design. Location: `internal/proc/proc_windows_test.go`. The Windows `BootTime` has no test, unlike its Linux counterpart. Evidence: `internal/proc/proc_windows.go` (diff 001, lines 97-101) and the existing `proc_windows_test.go` with no `BootTime` case. A row could check that the result lies in the past and that `ok` is true.
- F3. NIT, design. Location: `internal/shedverbs/loop.go` `runLoop`, the lock-failure branch. No row pins this branch's reason, "could not take the loop lock, so another loop owns the run", nor its `: <err>` suffix. Evidence: `TestUntilStop_LoserWaitsOnWinner` (`loopwait_test.go:1072-1078`) reaches the branch but asserts no mark. A row could add the mark check there, since the loser already takes this path.

Tool calls:
1. grep of `TestUntilStop_LoserWaitsOnWinner`'s assertions, the disagreement assertions in `waitshadow_test.go`, and the presence of `proc_windows_test.go`. My context did not yet hold those test bodies or the file listing.
