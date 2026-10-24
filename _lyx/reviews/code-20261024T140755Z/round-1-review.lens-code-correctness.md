```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function and branch of `internal/gatecli/run.go` against concrete cases.

**Per-function list**

- `report`, `halt: e.ctx.Err() != nil`, timed-out tier:
  - The tier's timeout context is derived inside `runTier` (`tierrun.go:60`), so `e.ctx` stays live.
  - `halt` is false, the run counts as failed with exit 1, and the call goes on. Correct.
- `report`, empty-tier failure (`emptyTierFailure` sets exit 1): `halt` is false and it counts as a failed run. Correct.
- `report`, signal arrives during a run:
  - `catchTerminatingSignals` stores the number before it cancels, so `caught` is nonzero.
  - `halt` is true, and `exitAfterSignal` returns 128+n. This matches the old exit code.
- `report`, `runCtx` cancelled with no signal, run exit 0:
  - `halt` is true, `end.code` is 0 and `caught` is 0, so `runTest` returns 0 and skips the remaining runs. Not correct. This is finding 1.
- `report`, refusal (`result.refusal != ""`): returns code 1 with `halt` true, and the JSON envelope is the only stdout write. Correct for a refusal that is the first thing to go wrong; see finding 2 for one after a failed run.
- `runWithSlots`, `lease == nil`:
  - A busy hub gives SlotBusyExit with `halt`.
  - A cancelled context gives 1 with `halt`, which `exitAfterSignal` turns into 128+n.
  - Another acquire failure gives 1 with `halt`.
  - Correct, and the slots are released by `defer` before the next run acquires.
- `runWithSlots`, empty run (`empty` true): no slot is taken, and a pass gives `runEnd{}`. Correct.
- `runGreenAware`, `end == (runEnd{})`:
  - It records a pass only for exit 0 with no halt, so a failed tier never carries.
  - A tier that passes just as a signal lands (`halt` true) records no pass. That is conservative, not wrong.
  - A carried tier goes through `reportCarried` and `report`, exits 0 and does not halt unless the context is cancelled. Correct.
- `runGuard`, `guardStepRun` error:
  - It returns `runEnd{code 1, halt}`, so the call still ends as before.
  - It loses the earlier failures' code and closing line; see finding 2.
- `runTest` loop and plan order:
  - `planTierRuns` puts guard runs last, so one loop covers listed tiers first and then guard runs.
  - `guardRunOf` has no caller left; the only mention of it in the diff is the deletion.
  - The first non-zero `end.code` sets `code` and `firstFailed` once, which is the first failure in plan order. Correct.
- Closing line:
  - It prints only when `failures` is non-empty, to `request.stderr`.
  - Entries read `tierN (exit C)` or `guard tierN (exit C)`, in plan order, and `firstFailed` names the exit code's owner. Correct.
  - A call where every run passes prints nothing. Correct.
- `runLabel`, `runEnd`: pure helpers, correct.

**Findings**

1. **MEDIUM, design**, `internal/gatecli/run.go:140-147` and `run.go:411` (`report`).
   - `halt` is set whenever `e.ctx.Err() != nil`, but the exit code takes the signal value only when `caught` is set.
   - If `runCtx` is cancelled without a caught signal, a run that exited 0 halts the call and `runTest` returns 0 with the later tiers and guard runs unrun.
   - Cancelling the parent `ctx` handed to `runTest` is one way to get there, and it reports a green gate.
   - Before this change the later runs would have been tried, and each would have failed on the cancelled context.
   - Fix: when a halt comes from a cancelled context and the code is 0, return a non-zero code, or halt only for a refusal or an acquire failure.
2. **LOW, design**, `internal/gatecli/run.go:143-145`, `runGuard`, `runWithSlots`.
   - A halt after earlier runs failed returns only the halting run's code.
   - Examples: a prebuild refusal in a later tier, a `guardStepRun` error, or a busy hub on a later tier.
   - The first failure's exit code and the closing line naming the earlier failed runs are both dropped.
   - The result is exit 1 or 75 with no closing line, even though the output already shows a failed tier.
   - The decision record says only that these stops end the call at once, so this may be acceptable. A halt after a failure could still print the closing line, or keep the first failure's code.
   - Exit 75 is documented as "no failure, run again", so after a failed tier it is misleading, though a rerun does redo the failed tier.

Tool calls: none; the manifest already held the code needed.
