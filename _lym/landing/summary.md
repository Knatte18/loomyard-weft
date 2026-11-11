# fabricengine: test the anchored-hub stops of pull and push

Steps of `fabricengine` that name a `lym` verb in a way forward resolve the verb's directory through `lymVerbDir`.
In a hub anchored at a subdirectory, that directory is the subdirectory of the code worktree, not its root.
Only the root-anchored hub had tests for these calls, so a regression in the anchored case would pass unnoticed.
This change adds the missing anchored-hub rows.

## Changes

- `pull_integration_test.go`: `TestPull_UpstreamAnchorStopRunsInTheAnchoredDir`.
  Origin changes the weft's anchor marker, and `Pull` must return a `WeftStopError` wrapping `ErrAnchorChange`.
  Its way forward must name `lym fabric anchor task --set other` in `<code worktree>/backend`.
- `pushclasses_integration_test.go`: the push anchor-change stop now runs per hub anchor.
  The "origin carries another marker" case runs in a hub anchored at `backend` and asserts the full `lym fabric anchor … --set sub` instruction with that directory.
  The other two cases stay in a root-anchored hub.
  `newFailureFixtureAt` and `newPairFixtureAt` take the anchor; `newFailureFixture` and `newPairFixture` delegate with `.`.
- `failedpush_test.go`: a `weftConflictStop` row asserts the way forward merges at the records root
  and names `lym fabric push` in the anchored directory.
  The weft-dirty stop already had an anchored row.

## Design decision

The replay-conflict stop in `settleUnmovedWeft` (`pull.go`) was built inline, so a test could not reach it without a full conflicting pull.
It is now the method `weftConflictStop`, next to `weftDirtyStop`, with the same fields and wording.
`WeftConflictStopForTest` in `export_test.go` exposes it, as `WeftDirtyStopForTest` does for its sibling.
This is the only production change, and it does not alter behaviour.

Closes nothing: the board entry links no issue.

## To check by hand

- The pull path's upstream-anchor stop had no anchored-hub row (`pull_integration_test.go`).
  `TestPull_UpstreamAnchorStopRunsInTheAnchoredDir` was added to fix it,
  and no fresh review has seen the fixed result, so read that test for coverage.
- Nothing else was run by hand.
