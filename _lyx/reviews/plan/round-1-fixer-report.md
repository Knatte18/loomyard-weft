# Plan round 1 — fixer report

## Changes

- **F1** (fixed as proposed): card 22 (was 21) creates `TestAgentActivity_LiveLockWaitReadsAsActivity`, edits `internal/loomcli#agentActivityFor`, and its Intent makes a live wait record read as activity right after the verify marker, with a dead or absent record falling through to the shuttle reads.
  The ImpactSummary names the watchdog change.
- **F2** (fixed as proposed): card 7 creates `FastForwardNotFastForward`, and `FastForwardPair` no longer calls `Pull`.
  It refuses a merge record, fetches once, refuses a dirty worktree and a non-ancestor upstream, then resets through `ResetHard` and pulls the weft through `PullWeft`.
  `Fabric.Pull` left the Uses list, which now names `mergeRecordExists`, `warpWorktreeDirty`, `warpUpstreamSHA`, `ResetHard` and `PullWeft`.
  The integration test gains the force-pushed-upstream row.
  Card 12 maps the refusal to a Stuck naming `lyx loom resume`, and its `mergein_test.go` state list gains the case.
  Its `refusal-spec.md` sentence now says "each of Merge-In's parent-sync Stucks" instead of "five", so the new Stuck gets its row.
- **F3** (fixed differently): card 2 states that `Store.Acquire` lives in `internal/parentlock/acquire.go`, and its Intent adds the file to the guard's list and the wait to the background's What counts list.
  The Edit list gains `pattern/PATTERN-daemon-wakeups.md` and the member glyph `cmd/lyx#unboundedWaitFiles`, not the bare path `cmd/lyx/daemonwakeups_test.go` the fix proposed.
  The reason is that the plan lists member glyphs rather than bare Go file paths in Edit, as the review's own Rejected section notes.
- **F4** (fixed as proposed): `lyx loom insert-card 17 merge-review-rubric` added card 17, which creates the stencil and `LoomRubricMergeReview` and edits `entries`, `rubricMarkerAllowlist` and `TestLoomRubrics`, with card 18's former rubric paragraph as its Intent.
  Card 18 (was 17) lost those five targets and the paragraph, gained `plan:contracts/stencils#LoomRubricMergeReview` under Uses, and dropped "stencil" from its ImpactSummary.
  The insertion renumbered cards 17-25 to 18-26.
  I corrected each prose mention the tool reported: card 12's "card 17 puts it into the recipes" now says card 18, card 18's two "card 19" mentions now say card 20, and card 20's "unused since card 17" now says card 18.
- **F5** (fixed as proposed): card 20 (was 19) dropped the engine-cap item and keeps one Publish moved case, the upstream push.
  Card 12 replaced the lock-wait case with an assertion that the lock is held before `syncParent` runs.
- **F6** (fixed as proposed): card 7's integration test gains the `FastForwardMergeInProgress` row, with a merge record from an unresolved conflicted fabric merge and the pair left unmoved.
- **F7** (fixed as proposed): card 1's `ForceRelease`/`ReleaseAll` bullet gains the stated bound.
- **F8** (fixed as proposed, renumbered): card 16 now says "which card 19's `New` reads".
  The review wrote card 18, but F4's insertion moved the off-switch card to 19.

## Disputed

None.

## Deferred

None.
