```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: whether each guard, refusal, branch and test row the plan adds can be reached, and whether a guarded site recurs unguarded.

Findings:

1. **MEDIUM. Card 10 (D1): the start path still meets today's error.**
   - Only `Command` sets the receiver's `readopt` seams.
   - `StartFirstPass` builds its own receiver at `internal/battencli/cli.go:634` and gives it a `reconcile` seam but no `readopt` seams.
   - Its pre-check `startFirstPass` calls `preflightRunOwner` at `internal/battencli/cli.go:454` before it spawns the detached pass.
   - Take a run with a found status at a row past Worktree-Create whose task worktree is gone. That pre-check still runs `reconcileResume`. `originTargets` then refuses with "no pair worktree", so the pass that would re-adopt the pair is never started.
   - Two production callers reach this path: adopt's green start (`internal/adoptcli/green.go:251`) and `lym beam create` (`internal/beamcli/deps.go:64`).
   - Card 10 does not cover this site and no row tests it.

2. **LOW. Card 7 (D6): the empty-lease skip can hide the no-origin skip reason.**
   - Card 7 skips the remote delete "with nothing reported" when `leaseSHA` is empty.
   - On a records repository with no origin, `archiveWeftTip` returns a skip reason and no tip (`internal/fabricengine/archive.go:78`), so the lease is empty.
   - `removeWeftWorktree` sets `remoteSkippedReason` from its own `RemoteURL` check (`internal/fabricengine/weftwiring.go:223`). The card does not order the new empty-lease check after that check.
   - If the new check runs first, the no-origin branch is never reached. `TestRemove_NoOriginUnderRemoteReportsSkipReasonAndCompletesTeardown` (`internal/fabricengine/removeremote_integration_test.go:124`) asserts that reason.

3. **LOW. Card 5 (D7): the no-push branch of the synchronous pushes fires on any exclusive holder, not only a removal.**
   - `PushPairAnchored` and `PushAnchored` try the lock shared and push nothing when it is held exclusive.
   - After cards 5 and 8, `Add`, `DiscardTaskBranch` and the cleanup sweep's try also hold it exclusive.
   - A `commit-records` push (`internal/loomcli/wiring.go:213`) that runs during a re-adopting `Add` or a cancel's discard therefore returns nil and pushes nothing.
   - The card's Intent says this happens "when a removal holds it exclusive". The skipped commits stay local until the next push, so nothing is lost.

4. **LOW. Card 8 (D7 and Goal): one origin deletion of a slug's former-pair branch stays outside the per-slug lock.**
   - `lym fabric cleanup`'s warp sweep deletes task branches on origin (`internal/fabricengine/cleanupremotewarp.go:126`). No card puts that delete under the slug's teardown lock.
   - A new pair's `Add` can therefore interleave with it, which the Goal ("no removal or deletion of a slug's former pair touches a later run's branches") rules out.
   - D7's list names only the weft sweep, so this may be intended. The plan does not say either way.

5. **LOW. Card 12 (D2 Land restore): a run kept under the legacy `self` directory never finds its `done` candidate.**
   - Card 12 reads the child status path as `shedrun.StatusRel(lymcwd.NamedWorktree(hub, slug), SelfRunID)`.
   - `runSegment` (`internal/shedrun/resolve.go:30`) picks the legacy `self` segment by checking the gone pair's directory on disk. With the pair gone it always answers `<slug>`.
   - A child run recorded under `_lym/shed/self/` therefore never yields its `done` candidate. The stop then says no copy of the records is left, though the archive tag holds them.

Further tool calls:
- Grepped `internal/battencli/cli.go`, `wire.go` and `arm.go` for where `shedPaths.CommitStatus` is set, and read `internal/fabricengine/commit.go:170-195`. Needed to confirm card 12's `CommitStatus` is wired before the pre-run (it is, through `wire` from `armAt`); my context did not have the wiring site.
- Read `internal/battencli/wire.go:1455-1490` and grepped for the callers of `wire`, to confirm `shedPaths` is filled in `armAt` before PreRun.
- Read `internal/shedrun/resolve.go:15-50` and `lymcwd.NamedWorktree`, to trace card 12's child-status path for a gone pair; my context did not have `runSegment`'s fallback.
- Scanned `armAt` (`internal/battencli/arm.go:359-465`) for any need of the task worktree ahead of the pre-run that would make card 10's path unreachable; none found.
- Read `startFirstPass` and `StartFirstPass` in `internal/battencli/cli.go` and grepped their callers, to check whether another receiver reaches `ownerPreflight` without card 10's seams; this produced finding 1.
