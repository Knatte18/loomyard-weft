# Fixer report — plan, round 1

All seven findings are fixed; nothing is deferred.
The `approved:` key in `00-overview.md` is untouched, and no git command was run.

## Changes

- **F1** (cross-references): card 3 now says card 6 writes the watched marker; card 4 names card 8 as the caller of its reading, in the Intent and in the ImpactSummary; card 5 says card 7 replaces `noticeStep` and that the notices are cards 7 and 8.
- **F2** (bound of the driver's silence): card 3's Intent now states what a watched-rendered driver's silence lets through (stops unannounced when batten dies or the orch strand disappears after the render) and its bounds (the pid-liveness test at render, the duplicate-never-loss direction for an unwatched driver adopted later, the notices of cards 7 and 8).
  Card 6's closing sentence states the same loss in place of the pointer to the record's Open risks.
- **F3** (recipe-blindness): card 3 now requires both notify stencils to stay recipe-blind, drops "batten announces those stops" from the watched rule's wording, and adds `cmd/lyx/driverstencil_test.go` to its Edit list to extend the scan to the two stencils.
- **F4** (card 1 tests): the Tests list is now the delta over the existing compact tests, naming `TestCompact_CompletesOnBoundaryAfterEntryAndIdle`, `TestCompact_RestartWithBoundaryReadTypesNothing` and `TestCompact_TimeoutReturnsToIdleAndHoldsAutomaticTriggers`, and the `landBoundary` migration to the fake's `autoCompact` map.
- **F5** (card 5 tests): the Tests paragraph names the arm tests that stay and gain assertions, names `TestInnerRun_AwaitingPollsFoldIntoOneHistoryEntry` as rewritten, and keeps as new cases only those with no existing equivalent.
- **F6** (cards 7 and 8 tests): card 7 names the existing notice and wire tests it extends (`TestNotice_StateChangeNotifiesOncePerEpisode`, `TestNotice_StatusChangeStartsNewEpisode`, `TestNotice_RestartDoesNotRenotify`, `TestNotice_NotifyErrorDoesNotChangeOutcome`, `TestWire_NotifyQueuesOnThePrimesOrch`, `TestWire_NotifyWritesNothingWithoutAnOrchStrand`); card 8 names `TestNotice_QuietOnlyForRunningChildWithLiveDriver` as rewritten to the activity.
- **F7** (Uses): card 3's `proc_linux.go` Uses entry and card 10's `windowtarget.go` Uses entry are removed, so card 3 has no Uses field left.

## Deferred

None.
