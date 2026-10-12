# Card 11 — The webster verify gate never re-runs a hung step and names the hang

**Create:**
- `internal/websterengine/verifyhang.go`
- `plan:internal/websterengine#verifyHang` -> `type verifyHang struct`
- `plan:internal/websterengine#classifyHang` -> `func classifyHang(res verifytree.Result, log string) (verifyHang, bool)`
- `plan:internal/websterengine#TestVerifyGate_HungVerifyFailsAtOnceWithoutRerun` -> `func TestVerifyGate_HungVerifyFailsAtOnceWithoutRerun(t *testing.T)`
- `plan:internal/websterengine#TestVerifyGateStuckReason_HungReportNamesTheHang` -> `func TestVerifyGateStuckReason_HungReportNamesTheHang(t *testing.T)`

**Edit:**
- `internal/websterengine#newVerifyGate`
- `internal/websterengine#NewVerifyGate`
- `internal/websterengine#VerifyGateReport`
- `internal/websterengine#renderVerifyGateFindings`
- `internal/websterengine#verifyGateStuckReason`
- `internal/websterengine#TestRenderVerifyGateFindings`
- `internal/websterengine#TestVerifyGateReport_RoundTrip`
- `internal/websterengine/doc.go`
- `contracts/specs/refusal-spec.md`

**Delete:**
- `internal/websterengine#TestVerifyGate_TimedOutVerifyFailsAtOnceWithoutRerun`
- `internal/websterengine#TestVerifyGateStuckReason_TimedOutReportNamesTheTimeoutAndLog`

**Uses:**
- `plan:internal/gateslot#WatchdogLine`
- `plan:internal/gateslot#FindWatchdogQuit`
- `plan:internal/gateslot#BusyQuitWayForward`

**Intent:**
D5 and D6 in webster's plan-level verify gate.

`classifyHang`, in the new `verifyhang.go`, reads a failed result and its log and reports a hang as `verifyHang{Matched string, Stacks bool, Window string}`:
- a log carrying the watchdog line `gateslot.FindWatchdogQuit` finds matches it, with `Stacks` set and `Window` its window;
- a log carrying go test's `panic: test timed out after` or the Go runtime's `SIGQUIT: quit` dump header matches that marker, with `Stacks` set;
- a log line starting `signal: `, go test's report of a test binary ended by a signal, matches that line, with no stacks;
- otherwise an exit code of 128 or above, or an exit code of -1 with an empty `Detail`, which `verifyrun` returns for a shell killed by a signal, matches `exit code <n>`, with no stacks.
The markers are checked first, in that order, so a hang a marker names reads by its marker.
The watchdog line comes first because a watchdog quit's log always also holds the runtime's `SIGQUIT: quit` header, which the watchdog's own SIGQUIT causes.
`res.TimedOut` keeps its own path, `failTimedOut`, unchanged.
A shell that could not start (-1 with a `Detail`) and an ordinary non-zero exit below 128 are no hang.

The gate closure in `newVerifyGate`, after the first failed run and before the rerun, reads the log once and asks `classifyHang`; a hang fails the gate at once through `fail`, with the report naming it, so the re-prompt goes to the fixer and spends one attempt, never `notes.addFlaky`.
After a rerun that fails, the same check runs on the rerun's result and log, so an ordinary first failure whose rerun hangs reports the hang.
An ordinary failure is still re-run once and a rerun pass still records flaky identities.
`NewVerifyGate`'s doc list says a hung first run is not re-run.

`VerifyGateReport` gains `Hang string` (`yaml:"hang,omitempty"`), the marker or `exit code <n>` matched, `HangStacks bool` (`yaml:"hang_stacks,omitempty"`) and `WatchdogWindow string` (`yaml:"watchdog_window,omitempty"`).
A hang report also carries the failing identities the log parses to, the log path and the log tail.

`renderVerifyGateFindings` gains a hang branch ahead of the failing-identities one: the step was not re-run, because it hung; when `HangStacks` is set it says the step hung and its goroutine stacks are in the log, and otherwise that it was killed or exited with the matched code and names it.
For a watchdog quit it also names the window and says that a package that is CPU-bound but healthy needs `gateslot.BusyQuitWayForward`, which is the operator's to do, not the fixer's.
Then it gives the matched marker or code, the failing identities, the log path and the tail.

`verifyGateStuckReason` gains the hang case: `the verify command hung (<matched>) and was not re-run; its goroutine stacks are in <log>` with stacks, or `the verify command was killed or exited with <matched> and was not re-run, see <log>` without.
After a watchdog quit it appends `; the gate test's CPU watchdog quit a test binary busy for <window>: if that package is CPU-bound but healthy, ` followed by `gateslot.BusyQuitWayForward`, `, then ` and the told `reentry` step.

Tests:
- `TestVerifyGate_HungVerifyFailsAtOnceWithoutRerun` replaces `TestVerifyGate_TimedOutVerifyFailsAtOnceWithoutRerun`, keeping its two timeout rows and assertions, and adds rows on `gateFake` with no new field.
  First-run rows each fail once, and each log ends with `failingPackageLog` so it parses to a failing identity: a log carrying `panic: test timed out after 10m0s`; one carrying `SIGQUIT: quit`; one carrying `gateslot.WatchdogLine("/tmp/x/a.test", 42, 2*time.Minute)` and, after it, `SIGQUIT: quit`, as a real quit's log does; and one carrying `signal: killed`, each with `ExitCode: 1`; exit code 137 with a plain failing log; and exit code -1 with an empty `Detail`.
  Each asserts one verify call, a non-terminal failure, `report.Hang` naming the marker or code, `HangStacks` as above, `WatchdogWindow` `2m0s` on the watchdog row, the failing identity the log parses to, an empty `notes.Flaky()`, and findings saying the step was not re-run.
  A rerun row: an ordinary first failure whose rerun log carries `SIGQUIT: quit` makes two verify calls and reports the hang with the rerun's log path.
  The existing `TestVerifyGate_FlakyFailurePassesOnRerunWithNote` and `TestVerifyGate_FailedEvaluationWritesReport` keep pinning the ordinary two-run path.
- `TestVerifyGateStuckReason_HungReportNamesTheHang` replaces `TestVerifyGateStuckReason_TimedOutReportNamesTheTimeoutAndLog`, a table keeping its exact timeout row.
  It adds: a `SIGQUIT: quit` report naming the hang, the stacks and the log; a watchdog report, written as the gate writes one after a real quit, naming the window, `busy_quit_sec` and `"lyx config gate" from the prime`; and an `exit code 137` report naming the code and holding no "stacks".
- `TestRenderVerifyGateFindings` adds a watchdog hang report and an exit-code hang report, asserting "not re-run", the window and `busy_quit_sec` on the first, the code and no "stacks" on the second.
- `TestVerifyGateReport_RoundTrip`'s `want` sets the three new fields.
The `internal/websterengine` budget row stays, two tests replacing two.

`contracts/specs/refusal-spec.md`'s "verify gate exhausted" row gains the hang case (`PATTERN-refusal-way-forward`): a hung step, named by its marker or exit code, is not re-run and its reason says so; after a watchdog quit the way forward is to raise `busy_quit_sec` through `lyx config gate` from the prime for a package that is CPU-bound but healthy, then re-step the Webster row.
`websterengine`'s package doc, in the verify gate section that says a timed-out run is never rerun, states D5's hang set and D6's report.

**ImpactSummary:** the webster gate closure skips its rerun for a hung result; the report gains three fields, and the findings and stuck reason a new branch; only this package reads them.

**Commit:** 11: webster verify gate does not re-run a hung step
