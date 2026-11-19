---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/shedverbs/loop_test.go:1580
    summary: New exit diagnostics lack regression assertions for directory, lock and PID-write failures.
    fix: Extend TestLoop_EndWithoutAnEnvelopeIsMarkedInTheLoopLog with directory-creation, lock-acquisition and PID-write failures while keeping LogPath writable. Assert each reason in the log, no envelope and exit code exactly 1. Capture and assert the existing refusal row's exit code too.
    origin: lens:code-tests
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/proc/proc_linux_test.go:44
    summary: BootTime's failure contract and Windows implementation have no behavioral coverage.
    fix: Add test-controlled inputs at the platform read boundary so BootTime can be checked for unreadable, missing and malformed btime data and an exact whole-second result. Add a Windows test of the uptime subtraction and successful result. Keep these checks in the existing platform test suites and avoid production-duration waits.
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/shedverbs/loopwait.go:240
    summary: Boot-time ordering is reported as proof of the loop's death cause.
    fix: Change unmarkedExitCause to report the observed timestamps and possible causes without asserting that a reboot or uncatchable kill ended the loop. Update its contract, package documentation and existing dead-loop table expectations to preserve uncertainty, including equal timestamps and an adjusted wall clock. If definite reboot detection is retained, persist and compare a stable boot identity instead of inferring it from wall-clock ordering.
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/shedverbs/loopwait.go:221
    summary: An informational reconciliation mark suppresses the new unmarked-exit diagnosis.
    fix: Distinguish terminal exit evidence from informational loop marks in loopExitEvidence. Keep reconciliation information but append the unrecorded-exit explanation whenever no terminal mark exists. Add a row to TestUntilStop_DeadLoopStopsLoopExited with a starting mark and reconciliation mark but no exit mark, asserting the fallback diagnosis reaches the interrupt cause.
    origin: lens:code-correctness
---

### [BLOCKING] Cover every newly recorded failure path

**Location:** internal/shedverbs/loop_test.go:1580

**Issue:** The table exercises dead-PID refusal and failed envelope writing only.
The newly changed directory, lock and PID-write branches in `runLoop` have no assertions on their exit reasons.
Removing their calls to `endWithoutEnvelope` would leave the diagnostic regression unchecked.
This violates `PATTERN-test-speed`'s coverage requirement under the round's constraint severity rule.
The table also discards the CLI exit code despite the helper's exact exit-code contract.
That assertion gap accounts for the surviving return-value mutation and is consolidated here.

### [BLOCKING] Cover the boot-time boundary contract

**Location:** internal/proc/proc_linux_test.go:44

**Issue:** `TestBootTime` reads the live host and accepts a difference of up to a minute.
It cannot exercise the documented unreadable-file, missing-line and malformed-value results.
It also does not pin the whole-second result returned by the Linux implementation.
There is no test calling the new Windows implementation.
These are uncovered behaviors under `PATTERN-test-speed`, rather than a request to rerun a mechanical gate.

### [MEDIUM] Preserve uncertainty in death attribution

**Location:** internal/shedverbs/loopwait.go:240

**Issue:** The new branches turn timestamp ordering into a definitive cause of death.
Windows `BootTime` subtracts uptime from the current wall clock in `internal/proc/proc_windows.go:82`.
A machine booted at 08:00 with a loop started at 10:00 can later advance its clock by three hours.
The computed boot time becomes 11:00 without a reboot, yet the halt note says a machine stop ended the loop.
Even accurate timestamps cannot distinguish a kill followed by a reboot from a reboot that killed the loop.
The opposite branch likewise cannot prove an uncatchable kill from missing log marks.
Whole-second boot timestamps and sub-second starts provide no basis for treating equality as proven prior boot.

### [MEDIUM] Informational marks are not exit evidence

**Location:** internal/shedverbs/loopwait.go:221

**Issue:** `runLoop` can append a reconciliation mark before stepping at `internal/shedverbs/loop.go:253`.
`loopLogLinesOf` includes every line carrying that loop's prefix in its marks.
If the loop and keeper then die without recording their exits, the nonempty-marks branch wins.
The halt note labels the reconciliation message as the loop exit and never calls `unmarkedExitCause`.
The requested diagnosis therefore remains absent on this path.

## Focus conclusions

Early returns now attempt loop-log marks; failed writes still produce trace warnings.
Stepping panics retain their interrupted-envelope path.
Unrecovered hook panics reach the keeper's process status and crash output.
No additional defect was established in those paths.

The Linux reader returns unknown on unreadable or unparsable input.
Windows uses uptime through `DurationSinceBoot`; the attribution defect is its use as causal proof in F3.
The review did not execute Windows code.

Older PID records decode with an unknown start.
Child updates retain the same record, and teardown restoration writes the complete record it read.
No dropped `StartedAt` consumer was found.

Held/background agreement matches the existing unawaited-shell hold and expiry paths.
Held/busy-turn remains a disagreement, and the loop's deadline and liveness checks still apply.
Removing this false warning satisfies the requested self-resolution alternative.
The diff's changes trace to the board entry's goals; no unrelated scope was found.

## Validation

Reviewed the committed range from e6b40df6aab7b28ad215edd407c21cf6775e466d to HEAD.
All lens reports are saved beside this report.
The focused tier-0 gate could not acquire a slot because the shared gate directory was read-only.
No tests ran in this review; the supplied gate reports were treated as evidence with their stated limits.
