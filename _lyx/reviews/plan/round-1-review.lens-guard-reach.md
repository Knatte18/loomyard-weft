```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: each guard, refusal and branch the cards add, traced from a real entry point. Read-only; no changes made.

## Per guard: reachable or not

- **Card 5, draft rule.** Reachable. Every `Board` write passes `boardCriticalSection`.
- **Card 6, migration while `draft_status` is absent.** Reachable on the first write after the binary lands. A `noWrite` result never saves, so the migration waits for a real write.
- **Card 7, draft start refusal.** Reachable through `armAt`, then `refuseAtStart`, then `startGuard.refuse`, for `run` or `step` with no status file. `StartFirstPass` arms through `armAt` too, so no start path skips it.
- **Card 13, state change between passes ends the Call.** Not reachable as written. See finding 1.
- **Card 13, done child under a covering pause yields.** Reachable: `runArm` reports the hold and returns nil.
- **Card 14, spawn hold under a pause or a standing owner.** Reachable. The owner re-check runs only when no pause covers the slug, so it fires only after the owner pause failed or was lifted. See finding 5.
- **Card 15, respawn cap.** Reachable. Each pass is a new pid, so the spawn reads as unconfirmed and a dead driver respawns.
- **Card 16, revive before the API-error notice.** Reachable.
- **Card 17, records-stop line.** Reachable when `Pull` returns a non-empty `recordsStop` (`land.go:79`).
- **Card 18, router stays silent for a pause-honoured Land stop.** Reachable only at hand-back, after the tackler and, with cards 24/25, the arbiter have already run. See finding 2.
- **Card 18, `RowStopNotice` once per stop.** Reachable.
- **Card 19, `CheckOverride` refusals.** Reachable.
- **Card 4, negative `beam.yaml` value refused.** Reachable.
- **Card 24, batten refusals of a budget-spent or precondition-unmet stop.** Unreachable through the driver: `decideDrive` hands those back before the tackler (`driverules.go:212-216`). The card says so and tests through `Check`.
- **Card 24, batten "landing halt" refusal.** Not reachable. See finding 3.
- **Card 26, exit 77 at pre-run.** Reachable through `resumeRefusal`, `resumeOwnerClaim`, the owned arm of `refuseResumeUnreachable`, and `refuseStartOwnedElsewhere` from `ownerPreflight` or from `armAt` via the `resolvePersistentPreRun` abort.
- **Card 26, the `claimBoardEntry` source row.** Not reachable at pre-run. See finding 6.
- **Card 26, exit 78.** Reachable: `refuseCovered` in `battenPreRun`.
- **Card 28, `StartFirstPass` refusals.** Reachable: the done slug through `checkRunStatus`, the covered slug through the pause check.
- **Card 29, marker-write refusal.** Reachable (a directory at the marker path).
- **Card 29, marker removal.** Reachable only on a pass that reached origin.
- **Card 30, `refuseContinuePass`.** Reachable through `--continue-pass` on a found status. With no status file, `armAt`'s `refuseStartOwnedElsewhere` answers first.
- **Card 32, `ErrDobbyRunning` and the owner exits.** Reachable.
- **Card 33, `planPass` branches.** Reachable, including a held driver lock read as in flight.
- **Card 34, owned-elsewhere handling and `recheckOwnerPause`.** Reachable for runs dobby has marked. The same stale-pause state recurs on an owner pause InnerRun set itself. See finding 4.
- **Card 35, covered first pass versus refused start.** Reachable: `armSeed` leaves a seed but no status file.
- **Card 37, dobby-start step failure in `create`/`promote`.** Reachable through the seam.
- **Card 39, both adopt refusals.** Reachable. The upstream checks in `namedEntries`, the reconcile and `sharedStopText` pass for the named inputs.
- **Card 40, a dobby-driven run reads alive.** Reachable. `tenderengine`'s orphan judge is another reader of batten liveness that is not widened. See finding 7.

## Findings

1. **MEDIUM · design · card 13 (`childWait.begun` from the persisted episode).** The state-change Stuck between passes cannot fire as ordered. `Call` runs `p.observeState(status.State, baseline)` before it builds `w`, and `observeState` replaces the episode with the new state (`notice.go:239-246`). So `begun`, read from the persisted episode, already equals the new state. Even with `begun` taken earlier, `checkChild` skips the decode when the mtime equals the baseline and no probe is due (`innerrun.go`, `checkChild`). The change is then seen at most on a probe-due pass. Until then the new state's arm runs: a running-to-done child can return Done with no state-change history entry. Fix: the card should state that `begun` is read from the loaded record before `observeState`, and compare `status.State` with it in `Call`.

2. **MEDIUM · design · cards 24/25 against card 18 (`PauseHonouredPrefix`).** Land's pause-honoured stop is `blocked` with reason "pause honoured…" (`land.go:173-177`). `decideDrive` sends a blocked stop that is not budget-spent, precondition-unmet or `HandBack` to the tackler (`driverules.go:233`). Card 24 makes every other blocked batten stop eligible for the arbiter. So an operator's pause spawns a tackler and now an arbiter, and card 18's no-notice branch is reached only after both. Fix: batten's eligibility should refuse a stop whose reason starts with `PauseHonouredPrefix`.

3. **LOW · consistency · card 24 (`TestEligibility_BattenRule`, the landing-halt row).** `Check` computes `LandingHalt` through `landingshed.HandsBack`, which knows only Merge-In and Finalize (`landingshed/handback.go:15-22`). Batten's driver deps set no `HandsBack` (`battencli/drive.go:154-232`). No batten status can be a landing halt, so the row needs a loom producer name, a state a batten run never produces.

4. **MEDIUM · design · card 34 (`recheckOwnerPause`).** The stale-owner-pause lift from the Added-after-Discussion F4 entry covers only the owner pause dobby itself set on a marked run. An owner pause InnerRun sets inside a continue pass is the same kind of entry (`PauseOwnedElsewhere`, `OwnerCaller`), and card 34 sets no mark for it. Under card 14 the spawn hold yields with the pause reason and runs no owner check while that pause covers the slug. An adopt that lands in the same window strands that run on a manual step. Fix: `recheckOwnerPause` should cover every live owner pause of a run dobby drives (`innerrun_owner.go` `ownedElsewhere`; card 14 Intent).

5. **LOW · design · card 14 (`holdSpawnWhileCovered`).** The pause-lift trigger in `foldOwnerTrigger` (persisted `ownerCovered` and `!covered`) never runs on the spawn-hold path, because `Call` returns the hold's end before `check`. After a lift, a run with `ownedBy` set waits up to one `notice_probe_s` for the owner re-check.

6. **LOW · consistency · card 26 (`TestPassExitCode`).** The rows read "each owned-elsewhere refusal source wrapped by the pre-run reads 77", and the card lists `claimBoardEntry`'s lost claim among the sources. That claim is lost only inside the Worktree-Create row, never at pre-run, so its row models a state that cannot occur there.

7. **LOW · design · card 40 (`registeredBattenAlive`).** The widening is pairteardown-only. `tenderengine.orphanOwner` still treats a slug with no live `RoleBatten` record as an orphan candidate (`processes.go:115-124`). Between passes a dobby-driven run has no batten record, so `lyx tender reap` can read the slug's records as orphans when no worktree carries the slug, as after a teardown in progress.

8. **LOW · design · card 28 (`ensureParentDobby`).** D2 refuses adopting a run whose parent another machine owns, because no dobby here would drive it. A hand `lyx batten run <slug>`, or a `StartFirstPass` from `lyx beam land`, of such a run spawns the parent's dobby anyway. Card 32's owner check then exits that dobby at once, so the run is left undriven with no refusal or warning naming `lyx adopt <parent>`.

9. **LOW · design · cards 26/35 (first pass under a pause).** On a first pass, `armAt`'s `refuseStartOwnedElsewhere` pulls the Bolt and claims the prime owner, and `armSeed` writes the seed. Both happen before `battenPreRun`'s `refuseCovered`. A pause that begins between dobby's read and the pass leaves Bolt writes and a seed behind before the covered refusal (`arm.go` `armAt`, `battenPreRun`).

## Tool calls
- `grep observeState` in `internal/battenshed/notice.go`: card 13's `begun` claim depends on `Call`'s `observeState` ordering, which the manifest did not hold.
- `grep HandsBack` in landingshed, `pausedReason` in land.go, `HandsBack`/`StartCheck` in battencli: card 24's landing-halt row and card 18's pause-honoured stop depend on these, which the manifest did not hold.
- `grep RoleBatten` across production code: other readers of batten liveness beyond pairteardown, for card 40.
