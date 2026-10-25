```yaml
lens: row-validity
outcome: reported
```

**Row-validity lens: findings on the umbrella-anchor plan**

- BLOCKING — Card 4, `TestDobbyGeometry`: the existing `a-beam` row breaks and the card doesn't say so.
  The test hand-builds a prime at `AnchorRel: "sub"` over a temp hub with no markers, and both rows expect the `.lyx/beam/` directory under `sub` (`internal/hubgeom/dobbygeom_test.go:24`, `:46`).
  After card 4, `DobbyGeometry` reads the beam's anchor through `NamedWorktree`, which answers `"."` in that hub.
  The card says only that "the prime row stays at the prime's `sub`"; it must also change the existing beam row's expectation, or seed a Bolt marker of `sub`.
- BLOCKING — Card 1, `TestResolve_AnchorScenario`: the new step breaks the stale-marker step after it.
  The steps run in order over one checkout's marker state (`internal/lyxcwd/lyxcwd_test.go:63-64`).
  The card adds its two-worktree own-marker step after the subpath step, so the later "stale marker" step runs with own markers still present.
  An own marker now wins over Bolt, so the stale guard never fires and that step fails.
  The new step must remove its own markers, or run last.
- MEDIUM — Card 5, `TestUnwire_LeavesSiblingWorktreeUndirtied`: none of the planned rows fails on today's code.
  Today's census checks siblings at `l.AnchorRel` and strips only `/<l.AnchorRel>/<name>` and the bare name (`internal/fabricengine/junction.go:593`, `:660`).
  Row 2 (unwired pair at `.`, prime still wired at `.`) is already kept today, and `/sub/_lyx` is never in the strip set.
  Row 3 (unwired pair alone at `sub`) strips `/sub/_lyx` today too.
  D9 is therefore untested. A discriminating row needs a legacy bare-name line kept for a sibling at another anchor, or a census run with an `l` whose anchor differs from the unwired pair's.
- MEDIUM — Cards 12, 18 and 19: the fake deps builders the new rows depend on are not on the cards.
  - Card 12: `TestPromote` builds its deps through `promoteWorld.deps` (`internal/beamcli/promote_test.go:37-38`). Without an `Anchors` filler there, every existing row dereferences a nil func.
  - Card 18: `TestCreate` needs `createWorld.deps` to record `AddOptions.Anchor` and the `EnsureAnchor` calls (`internal/beamcli/create_test.go:52-53`).
  - Card 19: `TestAdopt` needs `world.deps` with a `MoveInProgress` fake and its per-worktree state (`internal/adoptcli/adopt_test.go:88`).
  - Card 16 lists `fakeStart.deps` for the same need; these three cards don't list `internal/beamcli#promoteWorld.deps`, `internal/beamcli#createWorld.deps` or `internal/adoptcli#world.deps`.
- MEDIUM — Card 16, `TestEnsureLive` "a moving prime record refuses the prime and every beam": the row cannot fail if the propagation is wrong.
  The prime record reaches each beam only through `hubgeom.DobbyStart`'s `MoveInProgress` filler, which reads the worktree's own record and the prime's.
  The `fakeStart` row only sets a `moving` map, and no row of any test drives `DobbyStart`'s filler.
- MEDIUM — Card 9, `newDivergedWeftFixture`: it cannot build the planned "origin changed the marker while the weft has no local commit" row.
  The fixture always commits a local weft change (`internal/fabricengine/pull_integration_test.go:587`).
  The card adds only "an optional marker per side", not a way to skip the local commit.
- MEDIUM — Cards 10, 15, 17 and 18: their fixtures never make the target anchor a directory git tracks in the warp, which their checks require.
  `validateAnchorTarget` (card 15) and checkout's warp-tracked check (card 10) run `git ls-tree -d HEAD -- <anchor>`.
  `hubforge.ReanchorPair` (card 3) only creates the anchor directory when absent, and git doesn't track an empty directory.
  - Card 15: `TestAnchorChange` "moved from `.` to `sub`".
  - Card 17: `TestRunCLI_Anchor --set sub`.
  - Card 18: `TestAdd_AtAnchor` `y`.
  - Card 10: `TestCheckout_RefusesAnotherAnchor`'s first row, which expects no warp-update clause.

  As written, each of these refuses with "not tracked" instead of reaching the branch it claims. Each needs a committed file under the anchor in the warp.
- MEDIUM — Card 17, `TestRunCLI_Anchor` (prime `--set`): the success path ends in a dobby start that fails in a test binary.
  The path runs `dobbyengine.EnsureLive`, which spawns through `SpawnDetached`; that returns `ErrSpawnUnderTest` in a test binary and is reported as `OutcomeFailed` with an error (`internal/dobbyengine/start.go:104-105`).
  The card doesn't say whether that failure fails the verb, so the integrating-prime row may not be able to pass.
- MEDIUM — Card 8, `TestReconcile_MigratesMarkerlessWefts`: "the commits' push is spawned" names no seam that observes a spawn under test.
  The census-makes-no-network-call assertion is left as "a `GIT_TRACE` file or the push notifier fixture", which is a research step.
  "A markerless prime gets its content's anchor" and "the prime re-anchored at `sub`" need `ReanchorPair` on the prime, which its card-3 contract (a pair's records worktree) doesn't cover.
  After card 6 every hubforge pair inherits a marker, and the card doesn't say how a markerless pair is made (a commit removing `.lyx-anchor`).
- MEDIUM — Card 17, `TestRunCLI_Anchor`: the child-pair row doesn't reach the criterion it stands for.
  The criterion says a child forked before its parent's change "still runs, and lands into the parent, whose marker stays unchanged".
  The row only commits records on the child's weft branch, so no landing is driven.
- LOW — Card 9, `TestPushFailureClasses` "a fast-forward push of a local re-anchor lands": the row can't fail under the card's own design.
  The check sits only in `retryWeftPush`, which runs after a rejection, and a fast-forward push is never rejected.
  It only guards against misplacing the check before the first push.
- LOW — Card 7, `TestAdd_DropsParentRunRecords_SubpathAnchor`: the new "parent re-anchored" row needs run records committed in a pair parent's weft at its own anchor.
  The test's helpers commit them only through the prime's Location (`internal/fabricengine/add_runrecords_integration_test.go:215`).
  The helper change isn't stated, and the row needs `--parent`, which the test doesn't use today.
- LOW — Card 16, `TestRun_NoPassWhileAMoveStands`: the harness it runs on is `newLoopHarness`, whose default deps set no `Runs` or `SpawnPass` (`internal/dobbyengine/loop_test.go:53-80`).
  The card names `TestRun_LockOwnerAndWakeups` under Uses rather than `internal/dobbyengine#newLoopHarness`, and a due run must be faked for "the next tick starts the due pass".
- LOW — Cards 15 and 17: the rows needing a second machine's fetched upstream (the killed-integration re-run, and the second-clone integrate) don't list `internal/hubforge#CloneSecond` under Uses.
- LOW — Card 3, `TestReconcile_SweepsRetiredHubArtifacts`: the test already runs every case from the prime and from a pair (`internal/fabricengine/portals_sweep_integration_test.go:133`).
  The new case should be a third `from` value, a pair re-anchored at `sub`, rather than a separate row.

Tool calls:
1. `internal/hubgeom/dobbygeom_test.go` and the locations of `TestRegenerateHubWorkspace`, `TestReconcile_SweepsRetiredHubArtifacts` and `TestSpawnAnchoredScenario`: the fixture shape was not in context.
2. `internal/dobbyengine/loop_test.go` harness and the `Deps` docs: the tick test's fakes were not in context.
3. `internal/fabricengine/pushclasses_integration_test.go` `newPairFixture`, `droplocal_integration_test.go` and `buildReconcileFixture`: the push, drop-local and pull fixtures were not in context.
4. How the fakes are built in `beamcli` and `adoptcli`, plus the workspace and sweep test shapes: the fake constructors were not in context.
5. The fake constructors' bodies and the sweep test's loop header: needed to name the members and check the anchor loop.
6. The sweep test's `from` loop, `TestAdd_DropsParentRunRecords_SubpathAnchor`, and the creel test's hub shape: needed to judge repeats and fixture reach.
