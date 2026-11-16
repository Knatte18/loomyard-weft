```yaml
lens: bound-breaker
outcome: reported
```

Lens bound-breaker: I tried to break each bound, guard and code claim in the decision record with a concrete path or interleaving.

**Claims attacked**
- D1, constructor takes `gates` the way Webster's does: held. `websterEntry` allows only `gates` (`internal/shedrecipe/entries_simple.go:151`).
- D1, the guard stops an implementer switch with work half built: broken. See F2.
- D2, the arbiter spawn cap and the driver repair cap bound re-entry: broken. See F5.
- D5, attempt bound per batch (one per rung, D6 retries, D7 re-entry under the arbiter cap): broken through the same path as F5.
- D6, a machinery loss gets one same-rung retry: held as stated. Whether it is per rung or per batch is left to the interactions lens.
- D7, an arbiter fix is judged by the full post-batch checks against the start commit: held.
- D9, the per-batch commit sweeps in no other uncommitted work: held. `CommitRecordsPaths` stages exactly the `relPaths` it is given (`internal/fabricengine/commitweftpaths.go:21-41`).
- D9, Weaver never pushes: held, since the commit helper never pushes (`commitweftpaths.go:22`).
- D11, private pieces move to a shared package and Webster's behaviour stays: broken as a premise. See F4.
- D12, `FindRun` finds the live strand even when state never recorded it: broken. See F1.
- D13, a halted run's leftover strand is not woken and the re-entry carries the decision: broken. See F3.
- D13, renumber bound at the last begun card read from the active implementer: broken. See F6.
- D8, the arbiter description names the active implementer's run directory: held as a stated change. `arbiterRecipeDescription(recipe string)` takes only the recipe today (`internal/loomcli/arbiterprompt.go:35`), so the change has to add a settings read. Nothing contradicts it.

**Findings**

F1. MEDIUM, design. Location: D12.
- Summary: D12's crash guard rests on a false premise about `shuttleengine.FindRun`.
- Evidence: `FindRun(cfg, anchorPath, guid)` resolves a known strand guid to its run (`internal/shuttleengine/rundir.go:303-309`). It cannot find a run whose guid state never recorded, which is exactly the crash window D12 names.
- Webster's spawn-or-attach also attaches only through a recorded `prior.StrandGUID` (`internal/websterengine/recoverbatch.go:510`).
- Counterexample: the producer records attempt and rung, spawns the strand, then crashes before it records the strand. On the next call `FindRun` has no guid to look up, the gone-run path in D6 fires, and a second agent spawns on the same tree.
- Fix: D12 states how the attempt's strand guid is fixed and recorded before spawn, or which shuttle lookup by role or name finds it.

F2. MEDIUM, design. Location: D1 guard and D13 seam.
- Summary: "active" is never defined, so the guard fires too late.
- What happens: the guard runs only at the implementer row. D13's loom readers pick the "active" implementer, which in practice means `tapestry.implementer`.
- Counterexample:
  1. Weaver builds batches 1 and 2, then reroutes to Plan-Burler-Review.
  2. The operator sets `tapestry.implementer: webster` through `run_settings`.
  3. The plan gate and the rubric note now read Webster's absent state (`internal/loomshed/gates.go:254-276`). No card is frozen and the Plan-Review fixer rewrites done cards.
  4. Only afterwards does the Webster row refuse on Weaver's begun batches.
- A corrupt state file for the other implementer is not covered either. D1 does not say whether it refuses or reads as no batches.
- Fix: decide "active" by begun state, or refuse a switch at the readers too, and state what a bad or missing state file does.

F3. MEDIUM, design. Location: D13 decision-notice bullet against D12.
- Summary: D13 borrows a rationale that only holds for Webster.
- Evidence: `heldBackReason` holds back the notice on the grounds that "the fresh session the run's re-entry starts carries the decision" (`internal/loomcli/decision.go:437-448`).
- For Weaver, D12's re-entry attaches to the live in-flight strand instead of starting a fresh one.
- Counterexample: the run halts with a batch strand still live, and the parent records `lym loom decision add`. The notice is held back. Re-entry attaches to that same strand, which finishes the batch without ever seeing the decision.

F4. MEDIUM, design. Location: D11.
- Summary: D11 leaves open whether the shared pieces can leave `internal/websterengine` without Webster's state model, and the design rests on that answer.
- Evidence: the post-batch checks take Webster's `*State`, `*BatchState` and `Geometry` (`postBatchInputs`, `internal/websterengine/recordbatch.go:898-911`). `checkRecoveredSuspects` takes the same (`suspect.go:355`), and the fingerprint helpers take `*State` (`fingerprint.go:113-262`).
- Consequence: moving them means either moving Webster's `State` into the shared package or having Weaver keep Webster-shaped state. D3, D9 and D12 assume Weaver owns its own state file, so either way "Webster's behaviour stays as it is" is not shown.
- This is a deferred research question.

F5. LOW, consistency. Location: D2's Bound and D5's Bound.
- Summary: the re-entry bound names the wrong caps.
- Evidence: `driveRepairCap` counts re-steps of an interrupted stop only (`internal/sheddriver/driverules.go:21-22, 272`). It does not count a Stuck stop that re-enters the row.
- A Stuck stop also re-enters through the tackler's resume, under `tackler_max_spawns` (`internal/sheddriver/config.go:51-52`), and through a parent re-step. Neither bound names those.
- Counterexample: with `arbiter_max_spawns` at 0, each tackler resume still buys another last-rung attempt under D7, outside the cap D5 cites.

F6. MEDIUM, design. Location: D13 renumber bullet.
- Summary: D13 points the renumber bound at the active implementer's state but says nothing of the lease that makes the bound safe.
- Evidence: `runCardVerb` holds Webster's state-mutation lease around the begun-bound check and the renumber (`internal/loomcli/renumber.go:87-104`). Webster's `begin-batch` takes the same lease (`internal/websterengine/beginbatch.go:5`).
- No decision gives Weaver a matching lease over its batch begin.
- Counterexample: `lym loom insert-card` reads the last begun card as 3, and Weaver begins a batch over cards 4 and 5 at the same moment. The renumber shifts card 4 under a batch that has already begun.
- Fix: D13 names the lease both sides take.

Tool calls:
1. `internal/loomcli/renumber.go:80-130` and the callers of `AcquireStateMutation`: the manifest had only the grep lines, not the lease scope around the bound check.
2. `FindRun`, the driver's repair cap and the spawn-cap config: needed to test the D12 and D2 claims.
3. `FindRun`'s contract and `RecoverSpawnOrAttach`: needed to confirm attach needs a recorded guid.
