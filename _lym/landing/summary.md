# fabricengine tests: build live-state and merge layers once as templates

The `internal/fabricengine` tests rebuilt the same start states for every cell and test.
They now build each start state once as a registered `hubforge` template, and every test copies it.
The start state is the same by construction: each template's build is the fixture's former build steps, moved unchanged.
The change is test-only and exports nothing.
It speeds up the test pass.

## Live-state matrix

- `VerbCase` gains an optional `Build`, which holds the fixture steps a template build can run, such as adding a pair.
- `registerVerbTemplates` registers one template per anchor for each verb with a `Build`, keyed by `verbTemplateKey`.
  `runCell` takes its hub from `copyVerbHub`: a copy of the verb's template, or of the anchor's bare hub when the verb has no `Build`.
- `Arrange` now derives the verb's fixture from the copy's `Hub`.
  The Remove, Prune, Cleanup, Checkout and UnwireJunctions cases move their pair setup into `Build`.
  Cleanup and Checkout read the original branch from the copy.
- The Add case has no `Build`, so its `Arrange` stays whole.
  Its blocker is a single directory and no git step, so a template would only add a build.
- The Cleanup omission reasons now name `Build`, where the weft worktree is removed.

## Merge layers

- `newMergePairFixture` stays a plain copy for tests that need no scenario commits.
  It now delegates to `mergePairFixtureOf`, which opens the Fabric and the commit closures over a given hub.
- `newMergeLayerFixture` does the same over a copy of a registered layer.
- Six layers cover the branch-commit scenarios that at least three tests repeated:
  `conflictLayer` and `clashLayer` (a conflict on `conflict.txt` or `clash.txt`),
  `nonFastForwardLayer` and `warpProgressLayer`, `featureBothSidesLayer`, and `cleanMergeableLayer`.
  The conflict and non-fast-forward layers share the builders `conflictingFeature` and `nonFastForwardFeature`.
- The helpers a build calls (`commitOnBranch`, `branchAtCurrentHEAD`, `setupConflictingDivergence`, `setupCleanNonFastForward`) now take `testing.TB`.
- The merge, merge-recovery, merge-lock, merge-stage, merge-weft-local, mid-merge and weft-guard integration tests use the layers in place of their inline setup.

## Design decisions

- No build calls `os.Setenv`, `os.MkdirTemp` or `git config --global`.
  No build keeps the TB's `Context()` past the build, calls a package-global seam setter or a once-only registration, or reaches code whose result a once-only registration changes.
- An arrange that places a directory outside the hub stays whole in the test, since a build cannot place a path outside the hub.
- Tests that install a process-global seam take their copies before installing it.
- Tests that exercise `Add` still add their pairs.

## To check

- Run `lym gate test ./internal/fabricengine` for the integration tier.
  The assertions are unchanged, so the same tests should pass.
  No run of it is recorded for this change.
