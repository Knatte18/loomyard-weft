---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/battenshed/innerrun_binaryrestep.go:140
    summary: The parent's binary re-step logs a repeating seam failure at Warn on every due check, where the task's level policy requires one Warn per episode
    fix: >-
      In internal/battenshed/innerrun.go add failure-message constants beside failTacklerMarker for the re-step check (one per distinct seam, e.g. failRestepCheck keyed by what, and failRestepMarker, "battenshed: binary re-step marker unreadable; reading it as absent") and one for the Go verify-retry records ("battenshed: Go verify-retry records unreadable; leaving the halt to the usual handling"). In restepSkipped (innerrun_binaryrestep.go:140) replace the logger.Warn with p.seamFailed(failRestepCheck, err, "step", what), and call p.seamRecovered(failRestepCheck) once every seam of the check read without error (before the "if !changed" return and at the end of a clean pass). In restepRecorded (innerrun_binaryrestep.go:149) replace the logger.Warn with p.seamFailed(failRestepMarker, err) and call p.seamRecovered(failRestepMarker) when the read succeeds or the file is absent. In publishRetryDecision (publishretry.go:187, the GoVerifyRetrySpent error) replace the logger.Warn with p.seamFailed on its constant and call p.seamRecovered on a clean read. Extend TestInnerRun_ReadFailuresLogOncePerEpisode in innerrun_failures_test.go with rows for the re-step check seams and the Go verify-retry read, using the existing assertRepeatedFailureEpisode helper.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/battenshed/innerrun_binaryrestep.go:110
    summary: The dirty-pair skip logs the same Info on every due check while the halt, build and dirty paths stay the same, though the task's policy logs a wait or halt state at its change
    fix: >-
      In binaryRestep (innerrun_binaryrestep.go:109-112) log the "not re-stepping ... uncommitted paths" line at Info only when the dirty-path list differs from the one last logged for this halt and build, and at Debug otherwise. Keep that last-logged list on restepWatch and reset it where restepWatch is reset on a new stop key and on a changed build (innerrun_binaryrestep.go:63-65, 78-82).
---

### [BLOCKING] Parent's binary re-step Warns on every due check

**Location:** `internal/battenshed/innerrun_binaryrestep.go:140`, `:149`, and `internal/battenshed/publishretry.go:187`.
**Issue:** The task (commit "9: battenshed: log a repeating seam failure once per episode") put every polled seam in `battenshed` behind `seamFailed`/`seamRecovered` over `innerRunProducer.failures`, and `TestInnerRun_ReadFailuresLogOncePerEpisode` pins that.
`PATTERN-log-levels` and the `internal/logger` Level policy say the same: a failure that repeats each tick logs Warn once per episode, Debug while it persists, Info at recovery.
The parent's `binaryRestep`, wired into `haltedStep`, is a polled path that runs on every halted tick and checks on an interval that backs off to 30 minutes.
A persistent failure in it still repeats at every due check:
`restepSkipped` Warns on each one for the driver-strand read, the lock read, the build read and the dirty-path read, and `restepRecorded` Warns on each unreadable marker.
The parent's new `GoVerifyRetrySpent` read in `publishRetryDecision`, which runs on every probe while the child is blocked at a verify row, does the same.
Both sides compile and the tests pass because the new code never touches the task's `failures` set, so no test notices that it bypasses the policy.
The task's invariant (polled failures are episode-guarded) is broken by the parent's new code; neither side alone is wrong.
The `Warn` at line 125 on a refused spawn is bounded by `binaryRestepAttemptCap`, so it is not part of this finding.

### [LOW] Dirty-pair Info repeats on every due check

**Location:** `internal/battenshed/innerrun_binaryrestep.go:110`.
**Issue:** "Not re-stepping the halted child on the new build; the task pair has uncommitted paths" is an ordinary waiting state.
The Level policy logs such a state at Info at its change, but the line is emitted again at each due check while the halt, the build and the dirty paths are unchanged.
The interval backoff limits the rate but not the repetition.

## Rejected

- Shuttle: the parent's launch-carried skill load, `typeStartupColor`, `markStarted` and the `eventsProof` branch of `checkLivenessTick` add no per-poll logging, and the task's `run.failures` and activity episodes still guard the polled paths; the merged `skillReed` fake in `fakes_test.go` is the one the task's tests run against.
- Landed notice: `rowNotifier.ready` in `land.go` is the only landed-notice sender and its `readyMarkerLine` marker blocks a second send; the parent's `innerrun.go`, `notice.go` and `publishretry.go` changes add no landing path that sends or skips a notice.
- Pair teardown: the archive step in `RemovePair` runs after `EndSession` and before `remove`, so it sees the parent's added Go-driver quiet probe unchanged; `PATTERN-fabric-destruction-chokepoint` governs `package fabricengine` only, and the archive's `os.RemoveAll` lives in `pairteardown`.
- Patterns: the task's `reedengine.SpawnWatchdog` call site and the logger stderr discard under `go test` touch neither `PATTERN-driver-choice-single-site` nor the parent's other `PATTERN.md` changes.
- Orch stencil: the merged `orch-template-role.md` handles the `landed` notice (run loop item 4, Investigating a stop, After landing) beside the parent's go-driven addition, and `claims_test.go` carries both sides' claims without contradiction.
- Test budget: the merged `test-budget.yaml` rows equal the top-level test counts for `internal/battencli`, `internal/battenshed`, `internal/pairteardown` and `contracts/stencils`, which `cmd/lyx/testbudget_test.go` enforces.
