# Card 12 — innerrun-owner-check

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#NewInnerRun`
- `internal/battenshed#innerRunProducer`
- `internal/battenshed#innerRunProducer.Call`
- `internal/battenshed#innerRunProducer.check`
- `internal/battenshed#innerRunProducer.lockStep`
- `internal/battenshed#innerRunProducer.holdSpawnWhileCovered`
- `internal/battenshed#innerRunProducer.resumeAwaitingChild`
- `internal/battenshed#innerRunProducer.binaryRestep`
- `internal/battenshed#innerRunProducer.publishRetryResume`
- `internal/battenshed#lockHarness`
- `internal/battenshed#newLockHarness`
- `internal/battenshed/doc.go`

**Create:**
- `plan:internal/battenshed#OwnerVerdict` -> `type OwnerVerdict struct`
- `plan:internal/battenshed#ChildOwnedElsewhereError` -> `type ChildOwnedElsewhereError struct`
- `plan:internal/battenshed#ChildOwnedElsewhereError.Error` -> `func (e *ChildOwnedElsewhereError) Error() string`
- `plan:internal/battenshed#innerRunProducer.ownerStep` -> `func (p *innerRunProducer) ownerStep(ctx context.Context, w *childWait)`
- `plan:internal/battenshed#innerRunProducer.ownedElsewhere` -> `func (p *innerRunProducer) ownedElsewhere(ctx context.Context, w *childWait, owner string)`
- `plan:internal/battenshed#innerRunProducer.keepsLockForFinalize` -> `func (p *innerRunProducer) keepsLockForFinalize(status shedengine.Status) bool`
- `plan:internal/battenshed#TestInnerRun_OwnerCheck` -> `func TestInnerRun_OwnerCheck(t *testing.T)`

**Uses:**
- `internal/battenshed#newInnerRunDeps`
- `internal/battenshed#fakeClock`
- `internal/battenshed#newRestepHarness`
- `internal/battenshed#innerRunProducer.releaseLock`
- `internal/pauserecord#SleepWatch`
- `internal/pauserecord#Coverage`

**Intent:**
D6, inside InnerRun's wait, over told seams (`PATTERN-told-geometry`).
`InnerRunDeps` gains `OwnerCheck func(ctx context.Context) (OwnerVerdict, error)` (a judge-only run-scope reconcile; nil switches the check off), `PauseOwned func(ctx context.Context, owner string) error` (a tender pause of the slug carrying `pauserecord.OwnerCaller`), `ParentHoldsTaskWork func() (bool, error)` and `Restepped func(status shedengine.Status) (bool, error)` (whether the child's settle records hold a restep for that stop); `NewInnerRun` resolves nil `PauseOwned`, `ParentHoldsTaskWork` and `Restepped` to a no-op, false and false.
`OwnerVerdict` holds `Owner`, `Elsewhere` and `Unreachable`; `ChildOwnedElsewhereError` carries the `Owner` a spawn refusal named (card 14 builds it from kind `owned_elsewhere`).
The owner trigger's state lives on `innerRunProducer`, as `ownedBy` does, so it survives a `Call` boundary: its own `ownerSleep` watch, `ownerDue` and `ownerCovered`, the pause coverage of the previous check; `childWait`'s sleep watch keeps feeding the notices' idle clocks as today.
`check`, before `checkChild`, folds a wall and awake reading into `ownerSleep` and sets `ownerDue` when the watch records a new sleep (each at least `SleepTolerance`, a minute) or when `ownerCovered` was covered and this check's coverage is not, then records this check's coverage in `ownerCovered`.
`check` runs `ownerStep` after `checkChild` and before it returns any end `checkChild` gave on a status it read, over that status, while an end with no status read, a failed stat or a vanished status file, runs no owner step and leaves `ownerDue` for the next `Call`, so a sleep seen on the check where the child's state also changed still runs the owner check, and an `ownerDue` an unreachable verdict or a check error keeps is taken up by the next `Call`'s first check.
`ownerStep` runs on a check with `ownerDue` set: an unreachable verdict keeps `ownerDue` for the next check and holds nothing; a check error warns once per episode and keeps `ownerDue`; an owned-here verdict clears it and the producer's `ownedBy`; an owned-elsewhere verdict clears `ownerDue` and calls `ownedElsewhere`.
`ownedElsewhere` records the owner in `innerRunProducer`'s new `ownedBy`, releases the parent lock record naming its own child's run through `ChildParentLock.Release` (`releaseLock`) unless `keepsLockForFinalize` holds, calls `PauseOwned`, and sends one notice per episode naming the owner and `lyx adopt <slug>`; a `PauseOwned` error warns once per episode and is retried at the next notice probe while `ownedBy` stands.
`keepsLockForFinalize` holds when the child's status reads `CurrentProducer` Finalize with State running, or State failed with a non-empty `Transient` and `Restepped` false for that stop, and `ParentHoldsTaskWork` reads true; a read failure of `ParentHoldsTaskWork` or `Restepped` while the status reads either arm also holds, until the next check; any other state at Finalize, blocked or stopped included, does not hold, so its record is released.
`lockStep`, which keeps a covered holder's lock, now first releases its child's record when the coverage reports `OwnerHeld` and `keepsLockForFinalize` does not hold, with no reconcile of its own.
Every spawn on an existing run, in `Call` (the respawn of an unconfirmed running child), `resumeAwaitingChild`, `binaryRestep` and `publishRetryResume`, reads an error carrying `ChildOwnedElsewhereError` as an owned-elsewhere verdict: it calls `ownedElsewhere`, records nothing as spawned, writes no decision-acted, re-step or retry marker, and keeps waiting, never a failed spawn or a failed row.
While `ownedBy` stands, every spawn waits: `holdSpawnWhileCovered` also holds a spawn while `ownedBy` is set, re-running the owner check once per notice probe and ending the hold when it reads owned here, and the awaiting, re-step and retry arms skip their spawns, so a failed owner pause never makes InnerRun spawn and re-reconcile in a loop.
`Call`'s respawn refusal returns a budget-exempt Stuck naming the owner, whose re-entry meets that hold.
Bound: the child can act for up to one check interval after a wake before the check runs, and for as long as origin stays unreachable; a Finalize already past its parent-side merge keeps its lock and finishes its landing here, and once it is done its batten waits under the owner pause until `lyx resume <slug>` lifts it, then goes on to Land.
`doc.go` documents the owner check, the carve-out, the owner-pause release, the spawn refusal and the owner hold.
`lockHarness` gains fake `OwnerCheck`, `PauseOwned`, `ParentHoldsTaskWork`, `Restepped` and spawn-error fields, a pause coverage the row sets (covered, scope, `OwnerHeld`) in place of the fixed hub entry, and an awake offset a row advances apart from the wall clock to stage a sleep; `newLockHarness` wires them.
`TestInnerRun_OwnerCheck` is a table over that harness: a sleep of a minute or more runs the check once and a shorter one does not; a pause hold ending runs it; a sleep and a child state change seen on the same check run the check before that `Call` ends, and an unreachable verdict there leaves the next `Call`'s first check running it again; an owned-elsewhere verdict releases the child's record, pauses with the owner and sends one notice; an unreachable verdict retries at the next check and pauses nothing; a check error warns and retries at the next check; a `PauseOwned` error warns and is retried at the next probe; an `OwnerHeld` coverage releases the record with no check; a child at Finalize running with the parent holding the task's work keeps the record and is not paused out of it, as does a failed transient Finalize with no restep, while a failed Finalize with a restep recorded releases it and a child blocked at Finalize with the parent holding the work releases it; a `ParentHoldsTaskWork` or `Restepped` error at Finalize keeps it; a done child under an owner-held pause is held until the coverage clears and then ends Done; an awaiting child whose resume spawn returns `ChildOwnedElsewhereError` pauses with one notice, writes no decision-acted marker and spawns no more while the owner stands.
Two more rows run over the existing harnesses: a running child with no spawn confirmed whose respawn returns `ChildOwnedElsewhereError` returns a budget-exempt Stuck naming the owner and the next `Call` spawns nothing until the check reads owned here (over `newInnerRunDeps`); and a binary re-step and a Publish retry resume refused the same way pause, write no marker and do not spawn again (over `newRestepHarness`).

**ImpactSummary:** battenshed's InnerRun wait only; every new seam defaults to off, so batten's existing wiring keeps today's behaviour until card 14.

**Commit:** 12: batten: a live run re-checks its owner after a sleep or a pause hold
