# Card 12 — The loom verify gate names a watchdog quit and its way forward

**Edit:**
- `internal/loomshed#verifyFailureFindings`
- `internal/loomshed#NewVerifyGate`
- `internal/loomshed#TestVerifyFailureFindings`
- `internal/loomshed/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/gateslot#FindWatchdogQuit`
- `plan:internal/gateslot#WatchdogLine`
- `plan:internal/gateslot#BusyQuitWayForward`

**Intent:**
D6 in the loom verify gate (`internal/loomshed`), whose findings become the Webster-Burler-Fix row's and the Darn-Write row's halt reason when the gate runs out of attempts.
The watchdog fires in every verify command that runs through `lyx gate test`, and a watchdog quit of a healthy package has no code fix, so fixing the tests is not its only way forward.
This gate runs no flake rerun, so D5 changes nothing here.

`verifyFailureFindings` reads the whole log at `logPath`, not only its tail, and when `gateslot.FindWatchdogQuit` finds the watchdog line it adds a paragraph after the exit-code sentence.
The paragraph says the gate test's CPU watchdog quit a test binary that kept a CPU core busy for `<window>`, so its goroutine stacks are in the log: look there for a livelock or a spin, and if the package is CPU-bound but healthy, `gateslot.BusyQuitWayForward`, then re-step the row.
A log without the line renders as today.

`NewVerifyGate`'s doc line "`StatusFailed` fails with the exit code, the log path and the log's tail" gains: and, after a watchdog quit, the window and the `busy_quit_sec` way forward.

`TestVerifyFailureFindings` gains a `log` field holding each row's log content, defaulting to today's two lines, and one row whose log carries `gateslot.WatchdogLine("/tmp/x/a.test", 42, 2*time.Minute)` amid test output and ends with `last line`, which the loop asserts for every row, with `ExitCode: 1`.
That row wants the exit-code sentence, `2m0s`, `busy_quit_sec` and `"lyx config gate" from the prime`; the existing rows want no `busy_quit_sec`.

`contracts/specs/refusal-spec.md`'s rows "Webster-Burler-Fix verify gate exhausted" and "Darn-Write verify gate exhausted" gain the case: a failure whose log carries the watchdog line names the window, and raising `busy_quit_sec` through `lyx config gate` from the prime is the way forward for a package that is CPU-bound but healthy (`PATTERN-refusal-way-forward`).
`loomshed`'s package doc, where it describes the verify gate's findings, says the same.

**ImpactSummary:** one findings renderer in `internal/loomshed`, reached by the round and whole-diff verify gates of the Webster-Burler-Fix and Darn rows; a log without the watchdog line renders unchanged.

**Commit:** 12: loom verify gate names a watchdog quit
