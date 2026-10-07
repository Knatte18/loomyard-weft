# Round 2 fixer report

All ten findings are fixed in `_lyx/discussion/decision-record.md`; `_lyx/discussion/support-log.md` gained four Question-ledger entries.
No git command was run.

## Changes

- F1: Scope's verify-deadline bullet, the D3 heading and D3's Windows sentences now promise a process-group kill on Unix and a shell-only kill on Windows, with a log line on every Windows kill saying only the shell was killed; the Acceptance criterion matches.
- F2: the `PATTERN-shell-mechanics-seam` Constraint now keeps process-group setup and the group kill in `verifyrun`, in per-OS files using only `syscall`, with no `internal/proc` import.
- F3: D3 says the 60-minute deadline is a constant a test can shorten, and the `PATTERN-test-speed` Constraint carries it.
- F4: Goal and Scope narrow the "finished strand" promise to an ungated run's strand; D5's Bound says a gated run's strand is relaunched whatever its gate's state and states the finished-gated residual and its bound.
- F5: D2 says any failure of the mark op, at a shuttle site or in the landing callback, is logged and changes no verdict, error return or gate outcome; the Acceptance criterion covers both callers.
- F6: D7 waits for the bootstrap lock like `lyx loom start`, reads every input only after holding it and releases it on every path out.
- F7: D7 names the way forward of each refusal: `lyx loom start` for a missing status file, `lyx loom approve` or `lyx loom reject` for an awaiting run; a done run is a no-op success (support-log Q14); the refusal Constraint also admits a wait for a lock holder shown by `lyx loom status`.
- F8: D7 states that the shared resume branch and the reused refusal kinds name the calling verb, so `lyx loom resume` never tells its caller to run `lyx loom start`; the Acceptance criterion checks it.
- F9: D7 adds the halted-with-run-lock-held outcome and way forward, and orders the rows: a live unparked driver is refused without a merge probe, a parked or absent driver is probed for an unfinished merge first.
- F10: D6 revives only when reed state holds a non-live driver strand, so a Go-driven child gets no revive and no warn without reading the seed's driver (support-log Q16); its Bound says the pair-wide resume also relaunches gated and operator strands at their own prompts with no input typed.
  The `PATTERN-driver-choice-single-site` Constraint and the batten Acceptance criterion carry both.

## Deferred

None.
