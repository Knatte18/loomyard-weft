# Card 14 — The landing verify gate names a watchdog quit and its way forward

**Edit:**
- `internal/landingshed#verifyGate.verdict`
- `internal/landingshed#TestVerifyGate_CheckMapsVerifyResult`
- `internal/landingshed/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `plan:internal/gateslot#FindWatchdogQuit`
- `plan:internal/gateslot#WatchdogLine`
- `plan:internal/gateslot#BusyQuitWayForward`

**Intent:**
D6 in the landing verify gate (`internal/landingshed`), the post-merge verify and `publish_verify` of Merge-In, Publish and Finalize, whose Stuck reason is the halt reason.
These checks run no flake rerun, so D5 changes nothing here.

`verdict`, for a `StatusFailed` result that neither timed out nor failed to start, reads the log at `result.Log`, and when `gateslot.FindWatchdogQuit` finds the watchdog line it inserts a clause before the reason's way forward.
The clause says the gate test's CPU watchdog quit a test binary that kept a CPU core busy for `<window>`, with its goroutine stacks in the log, and that a package that is CPU-bound but healthy needs `gateslot.BusyQuitWayForward`, then resume.
The told way-forward clause, or the fix-forward clause, still follows.
An unreadable log or one without the line leaves the reason exactly as today, so the existing exact rows keep passing.

`TestVerifyGate_CheckMapsVerifyResult` gains a `logBody` field: when set, the row writes it to a temp file, builds a copy of its result whose `Log` is that path, hands that copy to `f.fake.result`, and makes every later assertion that reads the result's `Log`, the closing log-path check included, read that copy, so none passes on an empty path.
One row, "failed after a watchdog quit names the window and busy_quit_sec", sets `logBody` to output carrying `gateslot.WatchdogLine("/tmp/x/a.test", 42, 2*time.Minute)` and `ExitCode: 1`.
It wants `exit code 1`, `2m0s`, `busy_quit_sec`, `"lyx config gate" from the prime` and the temp log's path in the reason.

`contracts/specs/refusal-spec.md`'s rows "Merge-In: verify failed" and "Publish: verify failed" gain the case: a failure whose log carries the watchdog line names the window, and raising `busy_quit_sec` through `lyx config gate` from the prime is the way forward for a package that is CPU-bound but healthy (`PATTERN-refusal-way-forward`).
Finalize's plan verify reaches the same `verdict` but has no row today, so a new row "Finalize: verify failed" goes in the landing rows after "Publish: verify failed".
Its trigger is Finalize's plan verify, run after its catch-up merge-in, exiting non-zero; its class is correctness halt.
Its way forward is the told way forward when the failure record was written, and otherwise fixing forward on the task branch, then "lyx loom resume"; it carries the same watchdog case, the window and raising `busy_quit_sec` through `lyx config gate` from the prime for a package that is CPU-bound but healthy.
The row stays separate from "Merge-In: verify failed" because that row's way forward names the Webster and Darn gotos, which Finalize's reason does not tell.
`landingshed`'s package doc, where it describes the Stuck reason of a failed verify, says the same.

**ImpactSummary:** one unexported verdict in `internal/landingshed`, shared by the plan verify and `publish_verify` of all three landing producers; a log without the watchdog line gives the same reason as today.

**Commit:** 14: landing verify gate names a watchdog quit
