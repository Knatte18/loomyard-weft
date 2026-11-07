# fabricengine tests: live-state and merge-layer fixtures onto templates

The `internal/fabricengine` cross-product cells and merge tests now build their repeated start states once per test binary and copy them, instead of rebuilding them in every cell or test.
Only test files change; there is no production code change and no new export.

## Live-state cross product

- `VerbCase` gains an optional `Build`, which holds the fixture steps a hubforge template build can run.
  `Arrange` now derives the fixture value from the copied hub, and keeps only the steps a build cannot run.
- `registerVerbTemplates` registers one template per anchor for each case with a `Build`, keyed `fabricengine-livestate-<verb>-<anchor>`.
  `copyVerbHub` copies it in `runCell` and in `TestVerbCases_ArrangeSetsItsPreconditionField`, and copies the bare anchor shape for a case without a `Build`.
- Remove, Prune, Cleanup, Checkout and UnwireJunctions move their pair setup into `Build` unchanged, so each start state is the same by construction.
  Cleanup's `Arrange` reads the original branch from the Bolt checkout, which its `Build` never moves; before, it read the prime worktree's branch before the move.
- Pull, the UnwireJunctions hostile case and the CloneHub{Reset} non-hub target keep their `Arrange` whole, because each places a directory outside the hub.
- The live-state harness docs and the Cleanup omission reasons now name `Build` where they named `Arrange`.

## Merge-pair layers

- `newMergeLayerFixture` copies a registered layer and returns the same values as `newMergePairFixture`; both now share `mergePairFixtureOf`.
- Six layers, each repeated at three or more sites, move into registrations in `mergein_integration_test.go`:
  - `conflictLayer` and `clashLayer`: "feature" conflicts on `conflict.txt` or `clash.txt`, with "feature-weft" at the weft's HEAD.
  - `nonFastForwardLayer` and `warpProgressLayer`: a non-fast-forward "feature" with "feature-weft" at the weft's HEAD.
  - `featureBothSidesLayer`: `feature.txt` committed on "feature" and "feature-weft".
  - `cleanMergeableLayer`: the clean pair `newCleanMergeablePair` and the Merge lock test share.
- Sites whose branch commits differ, or that read state before the layer's commits, keep their own steps.
- `commitOnBranch`, `branchAtCurrentHEAD`, `setupConflictingDivergence` and `setupCleanNonFastForward` take `testing.TB`, so a build can call them.

## Design decisions

- `addCase` has no `Build`.
  Its arrange plants one directory and runs no git step, so a template would add a build and save nothing.
  This departs from the record's "each verb's arrange splits".
- `newMergePairFixture` itself stays a plain copy, as the record says; only the layers above it move.
- No build calls `os.Setenv`, `os.MkdirTemp` or `git config --global`, keeps the TB's `Context()` past the build, or calls a seam setter or once-only registration.
  No test that installs a process-global seam copies one of these templates.

## To check by hand

- Run `lym gate test ./internal/fabricengine` and confirm `TestCrossProduct`'s cell tally and the merge tests' assertions match the previous run; none was run for this description.
- `cleanup-original-branch` (design): in `cleanupCase`, confirm the Bolt checkout's branch equals the prime's branch before the move at both anchors, since the clean-state Effect asserts the primary weft branch survives.
