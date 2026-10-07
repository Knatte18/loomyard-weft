---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/battenshed/doc.go:26
    summary: Added or changed Go comments join two independent clauses on one line at a semicolon or a comma plus and or but, which PATTERN-comment-line-breaks requires to be broken
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/reedengine/strand_test.go:531
    summary: New fake-backed and scratch-tmux tests in reedengine skip t.Parallel and no comment names a global state that would justify it
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/loomcli/driverlaunch.go:125
    summary: Two behaviors the diff adds have no test, the reedDriverDirectory.DriverRow row selection and the wired ChildRunLockHeld probe in battencli
---

Range: `product.parent` is `main` and there is no `_lyx/loom/rework/` directory, so the range is `git diff $(git merge-base main HEAD)..HEAD`.
`go build ./...`, `go vet ./...` and `go test ./...` pass, as do `-tags integration` on `verifytree`, `verifyrun`, `loomshed`, `landingshed` and `battencli` and `-tags tmux` on the reedengine and reedcli wait-mark, strand-ops and unless-name tests.
`GOOS=windows go vet` passes on `verifyrun` and `verifytree`.

The plan's cards carry only `Edit`/`Create` lists, so there is no `Rename` or `Delete` group whose AST or `assert-no-callers` check could be missing.
`Marker.Note` and its test were removed by card 10, and a grep finds no remaining caller.

Verdicts on the focus directive:

1. Comment line breaks: no mid-sentence column wrap and no two sentences on one line in the named files.
   The wrapped-looking pairs a scan finds there are all breaks before a clause or a parenthesis, not column wraps.
   The independent-clause rule is not met, which is F1.
2. Test parallelism: every test the branch adds in `loomcli`, `battencli`, `battenshed` and `shuttleengine` calls `t.Parallel`, or is covered by a file header naming the global state it swaps (the logger output in `innerrun_halted_test.go`, the `isAlive` seam in `waitmark_test.go`).
   The gaps are in `reedengine`, which is F2.
3. Stuck reason of the verify gate: not a defect.
   `verifyGateStuckReason` names the timeout and the log path for a report with `TimedOut` and no `Failures` or `Dirty`, and `TestVerifyGateStuckReason_TimedOutReportNamesTheTimeoutAndLog` reaches it.
4. Revive episodes: not a defect.
   `callHalted` clears the revived marker whenever the halt-warned marker is rewritten, and `Call` clears it when the child is seen out of a halted or awaiting state.
   `AReHaltAtANewHistoryLengthRevivesAgain` covers a re-halt between two polls whose history length differs.
5. Redundancy in the `reedengine` strand tests: not a defect.
   The per-helper `strandNamed` test is gone, and the `Dormant` and `Hidden` rows of `TestAddStrandUnless_NamedStrandSkips` are the one engine-level assertion of those cases.
   The `//testtiming:keep` reason on `TestAddStrandUnless_NoNamedStrandAdds` still holds, since the add path's other covering tests run it without asserting the name or the persisted order.
6. Markdown line breaks: not a defect.
   Every changed line in `docs/overview.md` and the other changed `.md` files is one sentence per line.

### [BLOCKING] Comments join independent clauses on one line

**Location:** `internal/battenshed/doc.go:26-27`, `internal/battenshed/innerrun.go:448`, `internal/battenshed/deps.go:153,158`, `internal/shuttleengine/doc.go:64,147`, `internal/verifytree/doc.go:5`, `internal/verifytree/verify.go:70,133`, `internal/reedengine/state.go:36`, `internal/reedengine/lifecycle.go:161`, `internal/reedengine/windowsize.go:31`, `internal/battencli/wire.go:69,557`, `internal/landingshed/deps.go:161,165`, `internal/loomcli/start.go:132`, `internal/loomcli/driverresume.go:29`, `internal/loomcli/landingdeps.go:153`, `internal/loomcli/resume_test.go:120,127`, `internal/battenshed/innerrun_test.go:416,456`, `internal/testkit/shuttlefake/shuttlefake.go:214`, `internal/shuttleengine/fakes_test.go:59`
**Issue:** `PATTERN-comment-line-breaks` requires one sentence per line with a further break at an independent-clause boundary.
These lines keep a clause with its own subject and verb on the same line as the clause before it, joined by a semicolon or by a comma and `and` or `but`.
Example: `// ... once per halt episode per batten process; the run stays halted and nothing is resumed, so ...`.
The list is the instances a scan of the added comment lines found, and not a complete count.
**Fix:** Scan every comment line the branch added or changed for `; ` and `, and `/`, but `/`, or ` followed by a clause with its own subject and verb, and break each at that boundary.
Leave breaks before a participial phrase, a parenthesis or a list item, and leave untouched lines alone.

### [BLOCKING] New reedengine tests run serially without a named reason

**Location:** `internal/reedengine/strand_test.go:531`, `internal/reedengine/strand_test.go:582`, `internal/reedengine/waitmark_integration_test.go:17`
**Issue:** `PATTERN-test-speed` has a test call `t.Parallel` unless it touches process-global state, and a comment names that state.
`TestAddStrandUnless_NamedStrandSkips` and `TestAddStrandUnless_NoNamedStrandAdds` build an engine on `t.TempDir()` with a per-engine fake tmux, the same fixture the new `TestSetWaitMark` runs under `t.Parallel`.
`TestWaitMarkSegmentExpandsOnARealTmux` uses `newColdScratchEngine`, whose hub and socket derive from its own `t.TempDir()`, as the parallel `TestStrandOps_RealTmux` does.
None of the three touches env or cwd, and no comment at the test or atop the file names a global state.
**Fix:** Add `t.Parallel()` to the three tests and to each of their subtests that shares no mutable state with its siblings, and run `go test -race ./internal/reedengine/` and the tmux-tagged run to confirm.
If a test does touch process-global state, name it in a comment at the test instead.

### [MEDIUM] Two added behaviors have no test

**Location:** `internal/loomcli/driverlaunch.go:125`, `internal/battencli/wire.go:557`
**Issue:** `PATTERN-test-economy` and the rubric flag a behavior the diff adds with no coverage.
`reedDriverDirectory.DriverRow` selects the first row `loomengine.IsDriverStrand` accepts and passes a directory error through; `go test -cover` shows it at 0.0% under both the untagged and the `integration` runs, because `resume_test.go` fakes the whole `driverDirectoryProbe`.
The `ChildRunLockHeld` closure in `battencli/wire.go` creates the lock's directory, probes the run lock, releases a free probe and reports a held lock as true; the only test that references it checks that it is non-nil.
**Fix:** Add a case for `DriverRow` over a `reedDriverDirectory{directory: fake}` (no row, a non-driver row, a driver row, a directory error), and make the lock probe testable the way `driverStrandFrom` is, by extracting its body to a function over a lock path and testing a free path and a held one.
Extend the existing tests of those files rather than adding a new file.
