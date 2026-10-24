---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: "internal/gitexec/gitexec_test.go (TestRun_RemoteBounds) and internal/gitexec/gitexec.go (runAttempt)"
    summary: "the guard that leaves the stall check unarmed for a deadline at or under the stall bound has no test that pins it"
    fix: >-
      In TestRun_RemoteBounds, add a subtest that calls gitexec.SetStallBoundForTest with a value at or above the test deadline (the 1 s const set by SetRemoteDeadlineForTest),
      runs a silent ext remote through remoteAttempts,
      and asserts one run, one kill report, and a GitError with Timeout equal to the deadline and Stalled false.
      A guard changed to arm the check regardless of the deadline then fails it.
      Put it beside the other stall subtests.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: "internal/gitexec/gitexec_test.go (TestRun_RemoteBounds)"
    summary: "no test pins the acceptance criterion that a local subcommand gets no trace file and no stall bound"
    fix: >-
      In the existing local-command subtest that checks no low-speed environment, also assert that the local command's printed environment lacks GIT_TRACE_PACKET.
      In the subtest that checks a remote command gets the low-speed environment, assert that the printed environment holds GIT_TRACE_PACKET=.
      Add no new test function.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: "internal/gitexec/gitexec_test.go (TestRun_RemoteBounds)"
    summary: "trace file removal after a kill and retry, the single retry Warn and the override of an operator GIT_TRACE_PACKET are untested"
    fix: >-
      In the never-answers stall subtest, assert after the call that lyxtemp's run directory holds no git-packet-trace file,
      and, with the log capture helper the file already uses, that exactly one Warn was logged.
      Add the override assertion to the printEnv subtest: the child sees the lyx-chosen path.
---

Verdict: the diff makes the change the amended board entry asks for, within the decision record's scope, with no call-site changes.
The stall check fires once, at the stall bound, only while the trace file is empty, and is unarmed when `stallBound < deadline` is false.
The retry happens only after a stall kill, with a fresh deadline and stall bound, one Warn and at most one retry; a second stall returns a `*GitError` with `Stalled` set and `Timeout` set to the stall bound, which `IsTransportFailure` reports.
The stall kill is the deadline's own `cancel` through `ConfigureGroupKill`, and the stall callback checks `sawStall` before `ctx.Err()`, so a stall is never reported as a deadline timeout.
The trace file comes from `lyxtemp.CreateTemp` and is removed by a deferred `os.Remove` on every return path; only remote subcommands get `GIT_TRACE_PACKET`, a trace file, a timer or the pause hold.
`doc.go` describes the stall bound, the retry, the safety rationale and the override.
No `Pattern-Departure:` trailer is present or needed, and the diff declares no PATTERN change.
The three findings are test-economy gaps, each a behavior the diff adds with no test.

### [LOW] Unarmed stall check for a deadline at or under the stall bound is untested

**Location:** `internal/gitexec/gitexec_test.go:TestRun_RemoteBounds`, `internal/gitexec/gitexec.go:runAttempt`
**Issue:** The acceptance criterion "a deadline at or under the stall bound kills a hung remote once, at the deadline" is met only incidentally, by earlier deadline tests whose 1 s deadline sits far under the 30 s default stall bound.
No test sets the stall bound at or above the deadline, so the `stallBound < deadline` guard is unpinned.

### [LOW] Local subcommands getting no trace file is untested

**Location:** `internal/gitexec/gitexec_test.go:TestRun_RemoteBounds`
**Issue:** The local-command subtest asserts only the absence of the low-speed variables.
It does not assert that `GIT_TRACE_PACKET` is absent, and no remote subtest asserts it is present, though both are acceptance criteria.

### [LOW] Trace file cleanup, the retry Warn and the operator override are untested

**Location:** `internal/gitexec/gitexec_test.go:TestRun_RemoteBounds`
**Issue:** `PATTERN-lyx-temp-root` asks that the trace file be removed after the call and `PATTERN-log-levels` asks for one Warn per retry; the new subtests assert neither.
The package doc's claim that an operator's `GIT_TRACE_PACKET` is overridden is also unasserted.

## Rejected

- Trace-file creation failure fails every remote call with a bare wrapped error (lens:code-correctness): an unwritable `lyxtemp` run directory is an internal failure that already breaks other callers, and propagating it is the repository's stated error policy; a silent fallback to an unbounded call would hide it.
- "Empty trace" also holds while git waits on a credential helper, an ssh prompt or over `git://` (lens:code-correctness): the decision record's open risks accept the trace as the stall signal, lyx agents run non-interactively, and a `git://` remote fails safe by never counting as a stall.
- A stall timer firing in the same instant git exits non-zero would be retried (lens:code-correctness): the attempt had received no packet and changed nothing, so a retry is harmless.
- The `code-affected-set` and `code-pattern-fit` lenses found nothing: every reader of `GitError.Timeout`, every caller of the changed functions and every applicable PATTERN entry was checked and holds.
