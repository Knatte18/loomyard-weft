```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function or branch with one concrete input. I found no `design` defects. I made one tool call and ran no tests.

**Per function or branch**

- `registerVerbTemplates` and the `verbTemplates` init: correct.
  - Input: the `Verbs` table.
  - Only Remove, Prune, Cleanup, Checkout and UnwireJunctions have a `Build`.
  - Their names are unique, so the keys `fabricengine-livestate-&lt;Name&gt;-&lt;anchor&gt;` never collide.
  - Hostile cases such as `Remove/x` have no `Build`, so they are skipped.
  - Each registration is a fresh pointer, so hubforge's address-identity check holds.
  - `verbTemplates` depends on `Verbs` through the function body, so Go orders the initialisation correctly.
- `copyVerbHub`: correct.
  - With a nil `Build` it returns `CopyHub` on the anchor's bare shape, as before.
  - With a `Build` and anchor "." or "backend" it finds the template.
  - An anchor outside `anchors` would give a nil `Registration`, but `runCell` and the self-test only pass anchors that are present.
- `runCell` and the self-test: correct.
  - Both call `copyVerbHub` and then `Arrange`.
  - The CloneHub{Reset} loop at `livestate_matrix_test.go:382` still uses `CopyHub` plus `Arrange`. That is right today because those cases have no `Build`.
- Remove, Prune, Checkout and UnwireJunctions splits: correct.
  - `Build` holds the former pair steps, and `Arrange` returns the same `VerbFixture`.
  - The `slug` const is shared by `Build` and `Arrange` and carries no per-call data.
  - Every `Build` leaves the board HEAD equal to its upstream, because none touches the board.
- Cleanup split: correct.
  - `original` used to be read from `PrimeWorktree` before `AddPair`. It is now read from `BoardDir` after `Build`.
  - `BoardDirName` equals `BoltDirName` (`junctionnames.go:113`).
  - `primaryWeftBranch` documents the Bolt checkout as sitting on the warp's unsuffixed default branch, and nothing in `Build` moves it. So the value is unchanged.
  - Checking this against `RecordsBranchName(f.OriginalBranch)` at `livestate_verbs_test.go:666` gives the same primary weft branch as before.
- Checkout split: correct.
  - `OriginalBranch` is still read from `PrimeWorktree` after the worktree removals and before any branch switch, so it is the same branch as before.
- Add, Pull, Reconcile, hostile and CloneHub{Reset} cases: unchanged, and the exclusions are justified.
  - Add plants a blocker outside the pair.
  - Pull pushes to the code bare through a `tb.TempDir` scratch clone, which D7 keeps in the test.
  - The others add nothing a template could hold.
- `Build` bodies in `livestate_verbs_test.go`, no D7 process-wide call: correct.
  - They call only `AddPair`, `os.RemoveAll` inside the hub, and `mustGit` on hub paths.
  - They use no `os.Setenv`, `os.MkdirTemp`, global git config, `Context()` or seam setter.
- `mustGit` inside a `Build` (`livestate_helpers_test.go:15`, `livestate_verbs_test.go:625-636`): behavior differs from the old `Arrange` only in how a failure shows.
  - It panics instead of failing through the TB.
  - A `Build` runs on `buildTemplate`'s own goroutine, where `templateBuildTB` records failures and recovers nothing. A git failure there would therefore abort the whole test binary, not one test.
  - The same panic already happened in the test goroutine before, so this is a degraded failure mode and not a wrong result. I rate it LOW at most, class `consistency`.
- `mergePairFixtureOf`, `newMergePairFixture` and `newMergeLayerFixture`: correct.
  - The named-result `return f(...)` forms compile and return `(h, f, closures…)`.
  - `OpenFabric` now runs after the layer commits instead of before them.
  - Every test that moved reads the repository state when it acts, and none reads it earlier, so I see no dependence on the old order.
  - I did not read `OpenFabric` itself to confirm it caches no HEAD.
- Layer builds: each reproduces the replaced steps exactly.
  - `conflictLayer` and `clashLayer` run `setupConflictingDivergence` on the warp with "feature", then `branchAtCurrentHEAD` on the weft with "feature-weft".
  - `nonFastForwardLayer` uses ("warp-branch.txt", "warp-current.txt") and `warpProgressLayer` uses ("feature.txt", "warp-progress.txt").
  - `featureBothSidesLayer` and `cleanMergeableLayer` match the former `commitOnWarpBranch` and `commitOnWeftBranch` arguments, including `_lym/clean-weft.txt`.
  - All six registrations use anchor "." and are only copied through `newMergeLayerFixture(t, ".")`-equivalent sites.
  - None leaves the board unequal to its upstream.
- `commitOnBranch`, `branchAtCurrentHEAD`, `setupConflictingDivergence` and `setupCleanNonFastForward` (`t` → `tb`): correct.
  - Behavior is unchanged, and `*testing.T` still satisfies `testing.TB` at the remaining callers.
- Two copies of one layer, as in `weftguards` h1/h2: correct. Each copy is independent, and two registrations never share a key.
- Edited tests (`mergecrucible`, `mergein`, `mergein_recovery`, `mergelock`, `mergestage`, `mergeweftlocal`, `weftguards`): correct.
  - Each keeps the same assertions.
  - Order-sensitive steps are unchanged. In `TestMergeContinue_ConcludeFailureThenRetryConcludes` the hook is still installed after the fixture.
  - `TestMergeAbort_AfterConflict` still captures its start SHAs after the layer's commits.
  - In the crucible tests the `commitOnWarpCurrent` and `commitOnWeftCurrent` closures are still used and still returned.

**Findings**
- LOW, `consistency`, `livestate_verbs_test.go:625-636` (cleanup `Build`): `mustGit` panics inside the template-build goroutine, which `templateBuildTB` does not recover, so a git failure kills the test binary. Using `gitkit.Git(tb, …)` would report the failure through the build.

**Not verified:** whether a Prune copy relocates correctly when the pair's worktree directory is absent (a stale registration). Only a run of the tests would show it.

**Tool calls**
1. Read `livestate_helpers_test.go` (first 40 lines): my context lacked `mustGit`'s failure behavior.
