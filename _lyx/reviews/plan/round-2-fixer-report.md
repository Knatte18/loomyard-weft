# Fixer report — plan, round 2

## Changes

- **F1** (BLOCKING, fixed as proposed): `_lyx/plan/14-landing-watchdog-halt.md`'s Intent now adds a new row "Finalize: verify failed" to `contracts/specs/refusal-spec.md`, after "Publish: verify failed".
  The row's trigger is Finalize's plan verify after its catch-up merge-in exiting non-zero, and its class is correctness halt.
  Its way forward is the told way forward when the failure record was written, and otherwise fixing forward on the task branch and resuming.
  It also carries the watchdog case: the window, and raising `busy_quit_sec` through `lyx config gate` from the prime.
  I chose a separate row over widening "Merge-In: verify failed", because that row's way forward names the Webster and Darn gotos, and `verdict` (`internal/landingshed/verifygate.go:230-233`) gives Finalize no such clause.
- **F2** (MEDIUM, fixed as proposed): `_lyx/plan/07-pattern-tolerant-records.md` lists `internal/loomshed#writeRoundRecord` under **Edit:**.
  A new Intent paragraph says it writes through `state.WriteJSON` with lock path `recordPath + ".lock"`, which keeps unknown keys by D2 and still writes atomically.
  `*.lock` is already excluded from records git (`internal/fabricengine/recordsgit.go`'s `lockFilePattern`), and the existing `prrework` tests keep pinning the record's content.
  The ImpactSummary names the change.
  `lyx loom validate-plan` returned `ok: true` with informational findings only.
- **F3** (LOW, fixed as proposed): card 11's Wiring list now says `testRequest` gains a `stderr io.Writer` field, which `newCommand`'s RunE fills from `cmd.ErrOrStderr()`, and `runTest` passes `request.stderr` to the watchdog.
  `runTest` gets no new parameter, so it needs no re-sign.

## Disputed

None.

## Deferred

None.
