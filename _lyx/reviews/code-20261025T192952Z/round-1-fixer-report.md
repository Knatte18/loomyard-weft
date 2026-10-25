## Changes

- **F1**: fixed as proposed. `poll` in `internal/battencli/overview.go` drops the cached task worktree pair when the run is not in Run-Shed, so the next Run-Shed read resolves its marker again.
  The `pairs` comment now states the lifetime.
  `TestDobbyRunCells_ResolvesTheChildPairAgainInEachRunShedSpell` moves the pair's marker between two Run-Shed spells and asserts the second read uses the new anchor.
  Commit: `battencli: fix F1 — a task worktree's cached Location is dropped when the run leaves Run-Shed, so a moved anchor is read afresh`.
- **F1, self-check follow-up**: `poll` also drops the cached `childRecipes` entry in the same branch, and the test gains a third spell proving a seedless later run does not keep the first run's recipe.
  Commit: `battencli: fix F1 — the cached child recipe goes with the cached pair, so a later run's seed is read afresh`.
- **F2**: fixed with my own change, differing from the proposal in where the test sits.
  The proposal named a row in the `orchCLI` start test in `start_test.go`, but that harness builds the CLI by hand and never runs `PersistentPreRunE`, where the wiring lives.
  I split `Command` into `Command` and `commandOf(c *orchCLI)` in `internal/orchcli/cli.go`, and added the tier1 test `TestRunsWindowTitle_WiredFromTheWorktreeTheVerbRunsIn` in `runstitle_integration_test.go`.
  It runs the pre-run from a prime and from a beam over a forged hub and asserts the exact `runsWindowTitle` (`beam: prime`, `beam: <slug>`).
  No second test of `runsWindowTitleOf` was added.
  Commit: `orchcli: fix F2 — a tier1 test pins the runs window title wired from the worktree the verb runs in`.

- **Gate finding** (verify failure, `TestEnforcement_FabricVocabulary`: `internal/battencli/overview_test.go` imported `internal/weftname` outside its owner set): the F1 test now writes the hub-wide anchor marker under the board directory (`fabricengine.BoardDir`) instead of the worktree's own marker, so it imports no `weftname`.
  The test still moves the anchor between Run-Shed spells and passes; the `lyxcwd` guard test passes.
  Commit: `battencli: fix gate finding — the Run-Shed pair test anchors through the hub-wide marker and imports no weftname`.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency: LOW, `childRecipes` kept across a pair move; fixed (second F1 commit).
- fix-proposal: LOW, the same `childRecipes` case; fixed with the same change.
- fix-proposal: LOW, the status cache is keyed on modification time alone, so an anchor move with an identical status mtime would read stale; fixed differently: not changed, out of this finding's scope. A new run writes a fresh status file, and the cache is statuspane's, outside the finding's surface.
  Deferred, see below.
- fix-proposal: LOW, the F2 test runs the real `status` verb; disputed, premise false, see below.

## Disputed

- **fix-proposal LOW on the F2 test, premise false.** `status` restarts a watcher only when `tracked && strand.Live && !watcherLive` (`internal/orchcli/status.go:195`).
  The forged hub has no recorded strand, so `status` spawns nothing and the test has no side effect beyond reading.

## Deferred

- **fix-proposal LOW, `StatusCache.Poll` keyed on modification time alone** (`internal/statuspane/cache.go`): the only sound change is in `statuspane`, outside this round's write surface, and a moved anchor with an equal status mtime needs a status file that was not rewritten by a new run.
