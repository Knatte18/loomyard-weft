# Round 2 fixer report

All three findings of `round-2-review.md` are fixed, one commit each, and none touched `_lyx` or was pushed.

## Fixed

- **F1** (`a22f7d533`): the doc comments of `TestReflowSource_ReflowsDocComments` and `TestReflowSource_LeavesUntouched` in `tools/godocreflow/reflow_test.go` now put one sentence on each line, with no break inside a list.
- **F2** (`9d953ceb4`): the file header of `internal/selfreportengine/selfreport_test.go` now says the `CreateIssue` tests do not run in parallel because each swaps the package-level `NewGitHubClient` seam.
  The sentence names the `CreateIssue` tests only, because `TestDefaultLabels` swaps no seam.
- **F3** (`89e09ad6a`): `TestSeed_WritesExpectedStatus` in `internal/loomshed/seed_test.go` calls `t.Parallel()` at the top level, next to its parallel subtests.

Each package's tests pass after its fix.
No test was added: none of the findings is a coverage gap.

## Deferred

Nothing deferred.
