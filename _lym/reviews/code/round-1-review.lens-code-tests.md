```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests. For each behaviour the diff adds or changes, the test row that pins it, or none.**

| Behaviour | Pinning test | Result |
|---|---|---|
| `Record.Covers` (unfinished, subject listed, owner write not stood; `root` via `PrimeRecorded`, a beam via `OwnerStood`) | `TestWatchCovers`: done, root, stopped beam, prime recorded, owner stood | pinned |
| `Watch.Covers`: no record, undecodable record, unreadable board count as none | `TestWatchCovers` | pinned |
| `Watch.Covers`: a beam with no owner under an adopted `root` is covered, and a beam with its own owner is not | `TestWatchCovers` | pinned |
| `ownerSubject` resolving up a multi-level parent chain | none: every row's parent is `""` | unpinned |
| `parentOf` error path, and "unfinished record only, so no board read when there is no record" | none | unpinned |
| "Logged once per episode" for `Watch.Covers` failures | none | unpinned |
| Orch `ownerTick` keeps the strand under an adopt | `TestWatcher_StopsTheOrchOfABeamOwnedElsewhere`, new `underAdopt` row | pinned |
| Orch hold start/end logs, and the stop resuming when the adopt ends | none: the row asserts only `wantNoExit` | unpinned |
| `UnderAdoptWatch` and `orchCLI.underAdoptWatch` wiring, and the lazy build kept once with a build failure logged once per episode (`orchcli/watch.go`) | none | unpinned |
| Dobby adopt hold: no pass, no pause, no archive, no exit | `TestRun_ForeignOwner` row `AnAdoptInFlightKeepsTheDobbyUpAndIdle` | pinned |
| Hold skips the task start, tidy, test-map and `landBeam` steps (`loop.go` `scheduling` and `!d.adoptHolds`) | none: the row asserts only that no pass was spawned | unpinned |
| Hold start and end log lines ("adopt hold ended"), and the hand-over proceeding when the hold lifts | none | unpinned |
| Hand-over order (pause, archive, exit) and exit on success | `TestRun_ForeignOwner` rows 2 and 3 | pinned |
| Failed archive keeps the dobby up, retries at the next probe, then exits; the run stays paused | `TestRun_ForeignOwner` row 3 | pinned |
| Failed archive logged once per episode | none: one failure, and the test checks only that the line appears | unpinned |
| `pauseRunsForOwner` skipping runs that are done or already recorded owned elsewhere | none: only one running `r1` | unpinned |
| `pauseRunsForOwner` error paths (`Runs` fails, `ReadState` fails) | none | unpinned |
| Periodic Bolt pull at most once per interval, with the new owner acted on by the next probe | `TestRun_BoltPullBringsInANewOwner` | pinned |
| Failed Bolt pull logged once per episode, and the owner read going on | none | unpinned |
| `beamcli.pullBolt` turning a skip into an error | none | unpinned |
| `ArchiveOwnerTips` tags and pushes an unpushed code tip to the code origin, and a records tip to the records origin | `TestArchiveOwnerTips` rows 2 and 3 | pinned |
| `ArchiveOwnerTips` archives nothing when origin holds the tip | `TestArchiveOwnerTips` row 1 | pinned |
| `ArchiveOwnerTips` re-run is idempotent | `TestArchiveOwnerTips`, the re-run assertion | pinned |
| `ArchiveOwnerTips` unreachable origin gives an error wrapping `ErrArchiveFailed` | `TestArchiveOwnerTips` row 4 | pinned |
| Origin ahead of the local tip (`isAncestorOrEqual` path, nothing archived) | none | unpinned |
| Diverged origin branch, or branch absent on origin | none | unpinned |
| No origin remote, worktree missing, or detached checkout | none | unpinned |
| Teardown lock held, giving `ErrPairPushRunning` | none | unpinned |
| Several pairs: a beam pair's slug, and joined failures across pairs | none: only `root` with one pair | unpinned |
| "No branch push": the origin branch tip is unchanged after the archive | none: the test checks tag targets only | unpinned |
| `OwnerArchiveTagName` format, and that no reader of `archive/` tags (`ArchiveTagSlug`, `ArchiveTagPrefix`) takes it | `TestOwnerArchiveTagName` | pinned at the name level |
| Run report lists no run for a D11 tag (acceptance criterion) | none at report level | covered only through `ArchiveTagSlug` |
| `beamcli.archiveOwnerTips` pair assembly (own pair, `root` for the prime, a pair per `DobbyRuns` run) | none | unpinned |
| `ownerArchiveBeforeRefusal`: skip under adopt, prime slug mapping | none | unpinned |
| `archiveBeforeOwnerRefusal` in `lym beam start`: archive before refusing, failure joined with the retry hint, other refusals archive nothing | `TestStart_ArchivesBeforeRefusingAForeignOwner`, three rows | pinned |
| `createTagAt` and `pushArchiveTag` refactor | existing archive tests, not re-checked | pinned |

**Findings**

1. MEDIUM, design, `internal/fabricengine/ownerarchive.go:756-774` (test `ownerarchive_integration_test.go:25-37`).
   - The "only tips origin lacks" decision has no row for origin holding the tip as an ancestor or descendant. Missing rows are origin ahead (fetch, then `isAncestorOrEqual`, nothing archived), a diverged origin, and a branch absent on origin.
   - Add rows to `TestArchiveOwnerTips`.

2. MEDIUM, design, `internal/beamcli/ownerarchive.go:249-283` and `:289-306`.
   - Nothing tests the pair set the archive covers: the worktree's own pair (`root` for the prime), a pair per run, the skip under adopt, or the prime-slug mapping. A missing run pair or a wrong slug breaks D11 step 2 silently.
   - Add a test over a hub fixture that asserts the slugs handed to `fabricengine.ArchiveOwnerTips`.

3. MEDIUM, design, `internal/dobbyengine/loop.go:518-552` (test `handover_test.go:32-108`).
   - The hold row asserts only "no pass spawned". The D4 acceptance criterion says the dobby starts no task and runs none of D11's steps, but the task start, tidy, test-map and `landBeam` skips are unpinned.
   - Neither the hold's end log nor the hand-over resuming once the hold lifts is pinned.
   - Extend the hold row with spies on those steps, and add a hold-then-lift row.

4. LOW, design, `internal/orchcli/watch.go:874-926`.
   - `UnderAdoptWatch` (the `ReadOwners` and `ParentOf` wiring) and the lazily built accessor have no test, nor does the build-failure episode.
   - Add one wiring test.

5. LOW, design, `internal/orchengine/owner.go:46-55` (test `watcher_owner_test.go:31`).
   - The hold's start and end log transition and the stop resuming when the adopt ends are unpinned.
   - Add a two-tick row.

6. LOW, design, `internal/adoptrecord/watch.go:105-137` (test `watch_test.go:21-60`).
   - Nested parent chains, a `parentOf` error, and the log-once-per-episode behaviour are unpinned.
   - Add a nested-chain row (a beam with no owner under a beam with one), a `parentOf` error row, and an episode assertion.

7. LOW, design, `internal/dobbyengine/handover.go:362-372` and `:424-441`.
   - Unpinned: the failed Bolt pull episode and the continuing owner read, the `pauseRunsForOwner` skip of done and already-owned-elsewhere runs, and its error paths.
   - Add rows to `TestRun_ForeignOwner` and `TestRun_BoltPullBringsInANewOwner`.

8. LOW, design, `internal/fabricengine/ownerarchive.go:707-718` and `:695-702`.
   - The teardown-lock refusal and the multi-pair joined error are unpinned.
   - Add rows to `TestArchiveOwnerTips`.

9. LOW, design, `internal/fabricengine/ownerarchive.go:775-781`.
   - "No branch push" is asserted nowhere; the test never checks that the origin branch tip is unchanged.
   - Add that assertion to `TestArchiveOwnerTips`.

10. LOW, design, `internal/fabricengine/archivetag_test.go:53-68`.
    - The acceptance criterion "the run report lists no run for a D11 tag" is checked only through `ArchiveTagSlug` and the prefix. This is acceptable as pinned at the reader seam, so no new test is required unless the handler wants report-level coverage.

**Test economy:** no duplicate coverage found. `TestRun_BoltPullBringsInANewOwner` overlaps `TestRun_ForeignOwner` row 2 on the archive and exit assertions only incidentally.

**Tool calls:** none made; the diff and its test files were already in context.
