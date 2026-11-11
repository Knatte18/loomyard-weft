# Test the remaining lym verb directory calls in an anchored hub

In a hub whose anchor is a subdirectory, a `lym` step in a way forward must name the code worktree's anchored directory, not its root.
Two such steps had no anchored-hub test: the records-conflict stop of a weft pull, and the upstream anchor stop.
Every other way forward that names a `lym` step already had an anchored-hub row in `failedpush_test.go`, the weft-dirty stop included.

## Changes

- `internal/fabricengine/pull.go`: the records-conflict stop that `settleUnmovedWeft` returns is built by a new `weftConflictStop` method, beside `weftDirtyStop`.
  Its text and its fields are unchanged.
- `internal/fabricengine/export_test.go`: `WeftConflictStopForTest` exposes it for a `Fabric` holding only the two worktree paths.
- `internal/fabricengine/failedpush_test.go`: a tier 0 subtest asserts the conflict stop's exact way forward in the anchored fixture hub.
  The `git` steps name the records worktree, and `lym fabric push` names the anchored directory.
- `internal/fabricengine/pushclasses_integration_test.go`: the `OntoAnUpstreamCarryingAnotherMarker` row of the replaying pair push's anchor stop now runs in a hub anchored at `backend`.
  It asserts the `lym fabric anchor <slug> --set sub` step names `<pair>/backend`.
  `newFailureFixtureAt` and `newPairFixtureAt` build the fixtures with a given hub anchor;
  `newFailureFixture` and `newPairFixture` keep the root anchor.

## Design decisions

- The conflict stop gets a tier 0 test through a small extraction rather than an anchored variant of the diverged-weft pull fixture.
  That fixture's warp is a plain repository outside the hub, so it cannot reach an anchored directory without a new hub-backed fixture at tier 1.
  The existing tier 1 pull test still checks the stop end to end at the root.
- The upstream anchor stop is covered by moving an existing row to an anchored hub rather than adding a pair fixture.
  The root-anchored case of the same stop stays covered by the pull tests' marker-change rows.
