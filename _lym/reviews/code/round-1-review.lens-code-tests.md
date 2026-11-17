```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that would fail if it broke.

**Per behaviour, the test that pins it**

1. `leftoverKind`, the landed kind and its hold, pause and other-owner exclusions: `TestLeftoverKind` rows `Landed`, `LandedWithItsEntryOpen`, `LandedUnderABoltHold`, `LandedOwnedElsewhere`, `LandedCoveredByAPause`.
2. `leftoverKind`, cancelled without keep against cancelled with keep (entry removed too): rows `CancelledWithoutKeep`, `CancelledWithKeep`, `CancelledWithKeepAndItsEntryRemoved`.
3. `leftoverKind`, the hand-over kind under a hold and a pause, adopted back, and with a pass in flight: rows `HandedOverAndOwnedElsewhere`, `HandedOverUnderABoltHoldAndAPause`, `HandedOverThenAdoptedBack`, `HandedOverWithAPassInFlight`.
4. `leftoverKind`, the ended kind carried over from `strandedEnded`: rows `FailedWithItsEntryEnded…` through `MarkedOwnedElsewhere`.
5. `tearDownLeftovers` reads `listedRuns`, so done runs are included, and sends the kinds through the seam: subtest "a landed run's pair is torn down though batten reads done…".
6. Retry, backoff, ceiling, in-flight, unreported result and nil seam: the carried subtests of `TestTearDownLeftovers`.
7. Kept-pairs notice, once per fresh start and none on a re-exec: `TestKeptPairs` subtests 1 and 2.
8. The notice waits for `markKeptCancels`, which retries after a failure and logs once: `TestKeptPairs` subtest 3.
9. `BuildView` `Kept`: `TestKeptPairs` subtest 4.
10. Orphan notice and teardown no earlier than the grace: `TestOrphans` subtest 1.
11. An orphan with a commit inside the grace, or a live process, stays: `TestOrphans` subtest 2.
12. The first sight survives a restart: `TestOrphans` subtest 3.
13. The record drops a pair torn down or no longer an orphan: `TestOrphans` subtest 4.
14. An orphan read or record-write failure keeps the last reading, logged once: none.
15. `shedrun.WriteKeptCancel`: `TestRunIntents/KeptCancelWritesOnlyOverACancelledRecordWithNoIntent`.
16. `tenderengine` skips a pair with no deferred push: `TestResume` row "a lifted entry over a pair with no deferred push spawns no pair push".
17. `fabricengine.PairPushDeferred`, which reads the pending flag, and its `tendercli` wiring: none.
18. `battencli.MarkKeptCancels`, which picks cancelled runs with no intent whose pair stands, then commits and pushes: none.
19. `battencli.DobbyOrphans`, which classifies orphans (board entry, run directory, main and prunable skipped, the live-process read): none.
20. `fabricengine.PairLastCommit`, the newer of the two tips with a missing worktree skipped: none.
21. `beamcli.tearDownLeftovers` and `tearDownLeftover`, the per-kind dispatch (`Remote`, `Force`, `Landed`, local mode, orphan route, ended reconcile) and the workspace regeneration: none.
22. `pairteardown.RemoveHandedOverPair` and `RemoveOrphanPair`: none.
23. `battencli.DobbyLines` kept-pair line with its age and way forward: none (`TestDobbyLines` has no `Kept` row).
24. `battencli.DobbyRuns` new fields (`PairKept`, `KeptSince` through `keptSince`, `OwnedElsewhere`, `BoltHeld`, `HandOverMarked`): none.
25. `clearDoneRuns` skips a done run whose pair stands: none.

**Findings**

- **MEDIUM**, design. `internal/battencli/leftoverpairs.go:76` `DobbyOrphans`: no test pins orphan classification. A board-named pair, a pair with a batten run directory, the prime, or a prunable worktree classed wrong would be torn down after the grace. Evidence: `internal/battencli/leftoverpairs.go:90-117`; no `_test.go` names `DobbyOrphans` (grep).
- **MEDIUM**, design. `internal/battencli/leftoverpairs.go:30` `MarkKeptCancels`: no test pins the pre-upgrade selection (state cancelled, no intent, pair stands) or the commit and push. Acceptance criterion D7 "a pair kept by a cancel before the upgrade … is listed as kept after the first dobby start" is pinned only through the faked seam in `TestKeptPairs` and `WriteKeptCancel` in isolation. Evidence: `internal/battencli/leftoverpairs.go:38-66`; `internal/dobbyengine/leftover_test.go:255`.
- **MEDIUM**, design. `internal/beamcli/dobbydeps.go:270-327` `tearDownLeftovers` and `tearDownLeftover`: no test pins the kind-to-request dispatch (`Force` for cancelled, `Landed` for landed, local mode for handed-over, the orphan route, ended through the reconcile) or the regeneration of the hub workspace file. Acceptance criterion D7 "the hub workspace file no longer lists a pair after its teardown" has no test. Evidence: `internal/beamcli/dobbydeps.go:293,306-327`; no test names `tearDownLeftover` (grep).
- **MEDIUM**, design. `internal/pairteardown/leftover.go:18,34` `RemoveHandedOverPair` and `RemoveOrphanPair`: neither is tested. Nothing checks that the processes end before the local removal, or that an orphan's tips, uncommitted files included, are archived before removal and that a failed archive removes nothing. Acceptance criterion D7 "its branches archived" is unpinned. Evidence: `internal/pairteardown/leftover.go:18-48`.
- **MEDIUM**, design. `internal/fabricengine/pendingflags.go:46` `PairPushDeferred`, with its wiring at `internal/tendercli/deps.go:142`: no test pins that a left pending flag reads as deferred and its absence as not deferred. The acceptance criterion's "still pays a deferred anchor-change push" is pinned only against the fake `PushDeferred` seam in `TestResume`. Evidence: `internal/fabricengine/pendingflags.go:46-54`; `internal/tenderengine/resume_test.go:374`.
- **MEDIUM**, design. `internal/battencli/overviewdobby.go:206-212` `DobbyLines` kept line: `TestDobbyLines` has no row with `view.Kept`. Acceptance criterion D7 "listed with its age in lym beam status" is unpinned at the renderer, for both the age and the unknown-age wording. Evidence: `internal/battencli/overviewdobby_test.go:22-123` (no `Kept` match).
- **MEDIUM**, design. `internal/dobbyengine/donerun.go:117`: the new `run.PairPresent` skip in `clearDoneRuns` has no row. Without it the clear would remove a landed run's records before the leftover teardown reads them, and no done-run test sets `PairPresent`. Evidence: `internal/dobbyengine/donerun.go:117`; the only `PairPresent` outside `leftover_test.go` is at `internal/dobbyengine/loop_catchup_test.go:51`.
- **LOW**, design. `internal/fabricengine/orphan.go:28` `PairLastCommit`: untested (the newer of the code and records tips, a missing worktree reads zero). A wrong time would skip or wrongly allow an orphan teardown. Evidence: `internal/fabricengine/orphan.go:28-48`.
- **LOW**, design. `internal/battencli/dobbyreads.go:204-226,273` `DobbyRuns` new fields and `keptSince`: no test pins the reading of `PairKept`, `KeptSince`, `OwnedElsewhere`, `BoltHeld` or `HandOverMarked`. The engine tests set these fields by hand. Evidence: `internal/battencli/dobbyreads.go:204-205,223-226,273-282`.
- **LOW**, design. `internal/dobbyengine/orphan.go:628-654` `dueOrphans` failure path: no row covers an `Orphans` error or a record-write failure keeping the last reading, logged once per episode. Evidence: `internal/dobbyengine/orphan.go:629-653`; `TestOrphans` seams never fail (`internal/dobbyengine/leftover_test.go:302`).

Tool calls:
1. grep of `_test.go` files for the new identifiers: the manifest does not show whether tests outside the diff cover them.
2. Narrowed grep of those identifiers, plus `Kept` in `overviewdobby_test.go`: the first grep's file list was noisy and the renderer test was not in context.
3. grep for `PairPresent` in the dobbyengine tests and for pending-flag tests in fabricengine: to check whether the done-run skip and `PairPushDeferred` are covered.
