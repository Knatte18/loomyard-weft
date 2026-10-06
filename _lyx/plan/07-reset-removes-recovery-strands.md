# Card 7 — reset-removes-recovery-strands

**Edit:**
- `internal/websterengine/strand.go`
- `internal/webstercli/reset.go`
- `internal/webstercli/reset_test.go`
- `contracts/specs/refusal-spec.md`

**Intent:** D5, reset half: `lyx webster reset` removes every live recovery strand of the batches it resets, so no recovery agent keeps writing into a tree the reset just moved.

In `strand.go`, add `RemoveRecoveryStrands(reed shuttleengine.ReedOps, st *State) error`: for every recorded batch whose `Kind` is `"recovery"` and whose `StrandGUID` is set, in batch-number order, call `removeStrandIfLive`, which already logs each teardown; the first failure returns an error naming the strand guid.
A nil state removes nothing.

In the `reset` verb, call it after `PlanReset` succeeds and the standalone refusal has passed, and before `ResetPairWarp`, only for `--to start`, which resets every recorded batch; `--to pre-fix` resets only the verify gate's fixes, no batch, and removes nothing.
A failure refuses the reset with nothing changed: `webster: reset --to start refused: could not remove recovery strand <guid>: <cause>; way forward: run "lyx reed remove <guid>", then re-run "lyx webster reset --to start"`.
Update the verb's `Long` help to say `--to start` removes the run's live recovery strands first.

Add the refusal's row to `contracts/specs/refusal-spec.md`'s webster section, per `PATTERN-refusal-way-forward`: `reset: recovery strand not removed`, transient, with that way forward.

Bound: only strands webster's own batch state names as a recovery strand; implementer forks have no strand and the Master strand is never named there.

Tests in `reset_test.go`, through the cobra `reset` verb with a `shuttlefake` reed on the fixture's `websterCLI`: a `--to start` reset over a state recording one live and one dead recovery strand removes exactly the live one and resets; a `--to pre-fix` reset removes nothing; a removal failure refuses with the way forward, leaves HEAD where it was and saves no state.

**ImpactSummary:** Adds one websterengine helper and one call in the reset verb's start path; pre-fix resets and every other webster verb are unchanged.
