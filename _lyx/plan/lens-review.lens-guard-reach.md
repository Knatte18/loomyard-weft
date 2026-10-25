```yaml
lens: guard-reach
outcome: reported
```

Findings (guard-reach), on the 42-card umbrella-daemon plan:

1. **MEDIUM**: Card 26 (batten-pass-exit-codes), D4. A first pass with no status file that `refuseStartOwnedElsewhere` refuses exits 1, not `PassExitOwnedElsewhere`. That path is unguarded by the new code mapping.
   - Evidence: `internal/battencli/arm.go:395`. The check runs inside `armAt`, called from `resolvePersistentPreRun` (`internal/battencli/cli.go`). There an arm error is printed and `clihelp.Abort(ctx, 1)` is called, so `driveRun`, the only site card 26 routes through `passExitCode`, never sees the error.
   - D4 (decision record line 119) asks for "the same code" on this path.
   - `TestPassExitCode` calls `passExitCode` on wrapped errors, so no row covers the verb's real exit.
   - Card 35's refused-start handling still works, since it treats any non-zero exit as a refused start. The criterion "exits with its own code" is not met for this source.

2. **MEDIUM**: Card 18 (row-notices-after-chain), D8/D9. A pause honoured during Land's waits now notifies the orch as a Land failure. Today that halt deliberately sends no notice.
   - Evidence: `internal/battenshed#landProducer.pause` → `stuck(..., notify=false)`, which persists `blocked` with the "pause honoured" reason.
   - Batten's hand-back `Notify` in `internal/battencli#battenCLI.driverDepsFor` suppresses only `stop.Key().State == paused`.
   - Card 18 routes every non-paused Land stop's hand-back to `RowStopNotice`. A blocked "pause honoured" stop therefore reaches the tackler and then posts "<slug>: Land failed …".
   - The no-notice guard for the operator's own pause is not carried over to the hand-back path.

3. **LOW**: Card 10 (driver-yield-exit). The branch "a yield reached while resuming an open attempt is decided the same way" cannot be reached.
   - Evidence: `internal/sheddriver#goDriver.run` uses `resumeOpenAttempt` to resume a stop from the status or a saved attempt envelope.
   - A yield is never saved as an attempt, and a stop built from a status carries no `outcome`.

4. **LOW**: Card 15 (innerrun-respawn-cap). The card does not say when the child status file's modification time is recorded: "at the last respawn".
   - If it is read before `deps.Spawn` returns, the bootstrap's own status write after a respawn resets the count each time. The cap then never trips in production.
   - The planned test's fake spawn writes no status file, so it cannot catch this.
   - Evidence: `internal/battenshed#innerRunProducer.Call` spawn path. The intent should say the time is stored from a stat taken after the spawn returns.

5. **LOW**: Card 34 (dobby-owned-elsewhere). `recheckOwnerPause` lifts the owner pause when the local Bolt no longer names another machine. When the pass's owned-elsewhere verdict came from origin through `ReconcileOrigin` and the local Bolt was never brought level, the loop cycles on every probe tick:
   1. the local Bolt reads this machine;
   2. the pause is lifted and a full pass runs;
   3. the pass is refused as owned elsewhere;
   4. the run is paused again, and each pause interrupts turns after `ownerPauseGrace`.

   The card does not say which Bolt state follows the reconcile.
   - Evidence: `internal/tendercli#PauseOwnedElsewhere`; `pattern/PATTERN-owner-machine.md`: the verdict is judged against origin, and only an unreachable-origin resume reads the local Bolt.

Tool calls:
- grep for pause-coverage and `LockedCheck` sites in `battencli/arm.go`, `drive.go`, `shedverbs` and `sheddriver`: to check whether a continue pass meets any pause refusal beyond `refuseCovered`. My context did not hold the step and loop internals.
- grep of the hibernate and tender in-flight readers: to check whether another reader takes the gap between passes as "no batten".
- `hibernateengine/watch.go:300-325`: confirmed that after-all reads status files, not process records, so it is unaffected.
- grep of `loomcli/start.go` status writes: to judge whether the bootstrap rewrites the child's status file, for finding 4. It was not conclusive.
