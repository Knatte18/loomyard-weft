```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens review of the pawl-orch-landing plan: these are the forced edits that no card lists.

- **BLOCKING — Card 12 (D5, the move of `beamwiring.go`).** Card 12 moves `internal/loomcli/beamwiring.go` into beamcli, but its test stays behind. `internal/loomcli/beamwiring_integration_test.go:17` `TestBeamCommits_ListsTheBeamsOwnLine` calls `beamCommits` at `:41`. After the move loomcli's test package no longer compiles, and the tier-1 batch gate fails. Fix: add the test to card 12 as a second Move pair (`internal/loomcli/beamwiring_integration_test.go` → `internal/beamcli/describewiring_integration_test.go`), retargeting its package and fixtures.

- **BLOCKING — Card 15 (D3, `markReady`'s new refusals).** Card 15 makes `markReady` refuse an unflagged beam with no pawl run at Beam-Await, no stamped description and so on. Two integration tests call it on an unflagged hub fixture:
  - `internal/beamcli/dobbydeps_integration_test.go:178`, in `TestHubDobbyDeps_LandsOnlyAFlaggedBeam` (`:155`), calls `markReady(hubLandDeps(h.Location), "b")` and expects nil.
  - `TestHubLandDeps_MarkReadyCommitsAnUncommittedFlaggedRecord` (`:193`, call at `:208`) depends on its own setup.

  The batch gate (tier 1) fails on `:178`. Fix: list `internal/beamcli#TestHubDobbyDeps_LandsOnlyAFlaggedBeam` on card 15 and have it write the ready record directly. Also list `internal/beamcli#TestHubLandDeps_MarkReadyCommitsAnUncommittedFlaggedRecord`, or confirm its setup never reaches the new refusals.

- **LOW — Card 15 (the re-sign arrow).** `judgeReadyLanding` takes `ctx` and calls `ReadMergeState`, but `markReady(deps landDeps, slug string)` (`internal/beamcli/land.go:156`) has no context. Its callers are `readyCmd` (`cli.go:639`), `land_test.go:494` and `dobbydeps_integration_test.go:178/208`. The card either re-signs `markReady` with the arrow and lists those callers, or its Intent says where the context comes from (for example `context.Background()`). As written it does neither.

- **LOW — Card 7 (D2, `lym beam status`).** A member held by the ready-beam start hold shows in `lym beam status` as if nothing held it. `WaitingTask.ready()` (`internal/dobbyengine/view.go:66`) and the view build at `:117` read only `StartRefusal`, dependencies, ownership and pause. Yet `beam status`'s help (`internal/beamcli/cli.go:105-109`) promises to show what each unstarted released task waits on. No card fills a waiting reason for the ready hold (for example `StartRefusal` from `battencli.DobbyTasks`, `dobbyreads.go:326`) or updates the view.

- **LOW — Card 6 (D10, stale reopen text in the file headers).** The `land.go:1` header names `lym beam reopen` and the reopen request. The `deps.go:1` header names "the reopen request". Neither header belongs to a member card 6 lists (`landDeps`, `hubLandDeps`), so the Intent should name them as forced edits.

- **LOW — Card 12 (`PATTERN-fabric-git` / `PATTERN-gitrepo-client-boundary`).** The moved `beamCommits` brings a direct `internal/gitrepo` import into beamcli, where none exists today (`grep gitrepo internal/beamcli/*.go` is empty). That spreads a git-running site to a new package without saying so; the card should state it or put the read behind a fabricengine call.

Tool calls I made:
1. `lym quarry resolve` on 20 glyphs the cards name, to check spellings my context had not resolved. All were found.
2. grep for the callers of `ResolvePullRequest`, `resolveWithParentRule`, `beamCommits`, `beamParentBranch`, `NameBeamDescribe`, `beamUpRefusal`, `beamForkPrecondition`, `judgeLandOwners` and `markReady`. My context had no complete caller lists.
3. grep for the enforcement tests, the `ShedPaths` build sites, test uses of the `Reopen` field and stale reopen text in contracts and docs. These were not in the pre-read.
4. awk and grep for which function owns each `ShedPaths` site in `wiring.go`, the gitrepo caller test and the dobbydeps integration test names. My context lacked them.
5. grep for the import-allowlist tests (`seam_enforcement_test.go`) of loomshed, orchcli and shedrecipe, any reflection over dobby `Deps`, and the `StartRefusal` view code. Needed for registration checks.
6. grep for the test names in loomcli's beam wiring tests and stale Parent-Align text in pawlengine tests. Needed to confirm the beam wiring test's location.
