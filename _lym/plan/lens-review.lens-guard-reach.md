```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: whether each guard, refusal and branch the plan adds can be reached from a real entry point.

1. **BLOCKING (D10 bound / D11 entry, card 10): after a goto, the `recheckStanding` branch likely never runs.**
   - Card 10 reaches the branch only when Beam-Await's entry reads the board status naming the hold (Finalize or a retired row) with no land request.
   - On a goto the board status has likely already changed by then. Every shed persist that goes through `Shed.CommitStatus` writes the board status first: `newCommitStatusSeam`, `internal/loomcli/wiring.go:253-259`, calls `SetBoardStatus(boardStatus(producer, state))`.
   - `lym loom goto Beam-Await` leaves the run paused at Beam-Await. The resume's step 3b (the resume write, `internal/shedengine/run.go:303`) persists `running` at Beam-Await before the producer is called.
   - If that persist goes through `CommitStatus`, the board reads `running · Beam-Await` when `interruptedLanding` runs. The entry then finds no hold and takes the normal path: no land request, the ready record standing, the dobby stopped by the earlier accept, and the dobby's land episode recorded. The run waits at Beam-Await forever.
   - Only the crash case stays reachable. A status still `running` is not "resumed", so step 3b writes nothing.
   - Unconfirmed: I did not check whether step 3b uses `persist`, which commits, or `persistTransient`.
   - Card 10's `TestBeamAwait` rows fake `EntryProducer`, so they cannot catch this.
   - Fix: base the entry check on the shed history or the ready record, not the board status alone. For example: ready record standing, no land request, and the dobby-stop marker absent or the previous history entry naming Finalize.

2. **MEDIUM (D2, card 7): a member filed after ready but not startable gets no notice.**
   - `holdReadyBeam` posts only for "each released task `readyToStart` would start".
   - D2 says such a member "stays unstarted and blocks the landing, as any open member does; the dobby posts one notice per episode … naming it and `lym batten cancel <member> --remove`".
   - A draft member, or one held by a pending dependency, fails `released`/`readyToStart` (`internal/dobbyengine/starts.go:57-68`), yet `BeamLandRefusal` counts it as an open member.
   - The dobby's `landBeam` posts nothing when `Passes` is false (`internal/dobbyengine/land.go:41-49`).
   - Such a member blocks the landing with no notice from either path. No test row reaches this case.

3. **LOW (D12, card 17): a beam its own run holds at a retired row is still refused.**
   - `ownLanding` requires the producer to be in `LandingRows` (Finalize, Beam-Down).
   - Under the old binary, a pawl run halted at Parent-Align or PR-Review holds its beam: `RunHolds` treats any row outside `holdFreeProducers` as holding (`internal/boardengine/runlock.go:42,57`).
   - `lym beam start` therefore refuses the orch exactly when the operator needs it to follow the retired-row move.
   - D12 says "a beam its own pawl run holds, Finalize and Beam-Down included", which is wider than the two rows.
   - Fix: include the recipe's retired rows (`Deps.RetiredRows` or `loomrecipe.RetiredRows`) in what `ownLanding` accepts.

Tool calls beyond what my context held:
- Grep of `RowAt`, `Relaunch`, `arbiterSeam` and `finalizeGuard` in shedengine and loomcli: checked whether anything before `Step` refuses an unknown or retired row. Nothing does.
- Read of `internal/shedengine/goto.go:170-259`: checked whether `Relaunch` validates the current row. It does not without a Target.
- Grep of `loomArbiterSeam` and `HandsBack`: checked that the arbiter is wired for pawl runs, which card 9 relies on.
- Grep of `SetRunStatus` callers: looked for who writes the board status besides `pawlDeps`.
- Read of `internal/loomcli/wiring.go:170-214`: found the status commit seam's `SetBoardStatus`.
- Grep of `SetBoardStatus`, persist and step 3b: located the board write on each shed transition.
- Read of `internal/loomcli/wiring.go:225-269`: confirmed `newCommitStatusSeam` writes the board status on each changed transition. This took the call count to seven, one over the cap.
