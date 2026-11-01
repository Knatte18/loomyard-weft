```yaml
lens: bound-breaker
outcome: reported
```

Lens: bound-breaker. I tested each claim and bound in the decision record against the code.

**Findings**

1. **MEDIUM — D3, first bullet, and Open risks.** The claim that "dobby gives a covered run no full pass" does not hold. A run the dobby already marked owned elsewhere still gets a full pass under the hold.
   - Counterexample: before the adopt, a dobby on this machine is live, for example after an earlier `lym s` or because this machine owned the run before. It passed the run, the pass exited owned-elsewhere, and `ownedElsewhere` set `rs.OwnedElsewhere` and wrote an `OwnerCaller` entry. Adopt then turns that entry into the hold and writes the owner. On the next probe tick, `recheckOwnerPause` (`internal/dobbyengine/owned.go`) reads the owner as this machine. It calls `LiftOwnerPause`, which finds no `OwnerCaller` entry and returns nil. It still sets `liftPending[slug] = true`. In `planPass`, the `rs.OwnedElsewhere &amp;&amp; trigger.Lift` branch then returns a full pass whatever `covered` says.
   - The hold survives only if batten's first pass refuses a covered run (exit 78). The record should say that the hold rests on that refusal. Either `recheckOwnerPause` must set no lift trigger when no owner entry was lifted, or it must not lift while a `HoldCaller` entry stands. An acceptance criterion should pin this: an owned-elsewhere-marked run under a hold gets no pass after the owner write.
2. **LOW — D3, bound.** A covered run still gets continue passes. When a dobby here already drove the run at Run-Shed with no full pass owed, `planPass` returns `passContinue` while it is covered. Per PATTERN-owner-machine, a continue pass skips the pre-run pause refusal. So a batten process does run under the hold, and only its agent and network work wait at the pause gate. The record's "starts no pass" and "holds" wording says more than that. The bound should name continue passes as the allowed leak, or D3 should close it.
3. **MEDIUM — D5, `landed`.** `landed` is read as "this machine's records held not done before the reconcile, and the record reads done after it". `ReconcileOrigin` tears down the local pair of each run origin has ended, and that teardown takes the pair's records sibling, where the run record lives, before any branch moves. So after the reconcile the record is gone, not "done", and the run would be missing from the report. `landed` must be read from the reconcile's ended set or from the board status, not from the run record after the reconcile.
4. **LOW — D3, "a run whose pair is live here reports running_here".** Under `--keep-local` over a live pair whose subject was owned elsewhere, the run gets no hold. If its driver is dead while its strands live, the dobby that adopt's beam steps start gives it a first full pass at start. That run goes on without a go-ahead, which breaks the Goal's "starts no run" for that case. The record should state it as a bound, or hold the run anyway.
5. **LOW — D4.** `lym beam go` lifts the hold and then calls `StartFirstPass`. The dobby also answers the lift with its own full pass. The driver lock admits only one, so this is the same race adopt has today. Harmless, but it is unstated.

**Claims checked**
- D3, at most one live entry per scope, so an operator entry blocks the hold: held.
- D3, `lym tender resume &lt;slug&gt;` lifts an entry of any caller: held (lifts the pair's own entry).
- D3, the hibernate wake's `holdRuns` adds no entry over a standing hold: held.
- D3, the hold is written before the owner write, so a live dobby cannot pass the run: broken (finding 1).
- D3, covered means no pass: broken (finding 2).
- D5, the landed detection: broken (finding 3).
- D2, the reconcile covers every warp worktree, its records sibling and Bolt: held, from the context's reading of `originTargets`.
- D4, idempotent with nothing held: held.

**Calls made**
- Read `planPass`'s tail in `internal/dobbyengine/passes.go`, the hibernate wake's `holdRuns` and a grep of `tenderengine`'s resume: I needed the covered and lift branches and the wake path, which my context lacked.
- Grepped `internal/dobbyengine/owned.go`: to find dobby's owner-pause lift handling.
- Read `recheckOwnerPause` in `internal/dobbyengine/owned.go`: to confirm the lift trigger is set even when no owner entry is found.
