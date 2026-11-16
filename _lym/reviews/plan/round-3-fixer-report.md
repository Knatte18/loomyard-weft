# Round 3 fixer report — plan

## Changes

- **F1 (BLOCKING) — fixed, partly differently.**
  The premise holds: `commitWeftLocked` (`internal/fabricengine/recordsgit.go`) and the re-anchor commit write `Warp-SHA` into weft commits that are no transition.
  - Overview, `code-tip-is-warp-sha`: transition commits carry `Run-Transition: <run id>` beside `Warp-SHA`.
    The last transition's tip is the `Warp-SHA` of the newest commit whose `Run-Transition` names the slug.
    The rationale now says why the second key is needed.
  - Card 6: **Create** gains `TransitionTrailerKey`, placed in `trailer.go` beside `WarpSHATrailerKey`.
    `lastTransitionCodeTip` selects by that key, and step 1 no longer says "the newest commit's `Warp-SHA` trailer".
  - Card 7: **Uses** gains `TransitionTrailerKey`, and `TestOriginCodeTip` gains the row with a newer `lym fabric commit` style commit.
    The give-up sentence now says "an older transition commit's `Warp-SHA`".
  - Card 8: both `Commit` closures pass the `Run-Transition` trailer with `shedrun.ResolveRunID(location, runID)`, and **Uses** gains both symbols.
    The `TestCommitStatusSeam_Real` row asserts the `Run-Transition` trailer as well.
  - Card 12: `ReturnToPreviousTransition` reads its status at the previous `Run-Transition` commit, and its single-transition rule counts those commits.
    The rule now has the same wording in the bullet and in the paragraph.
  - Card 14: the `PATTERN-run-fsm` background now names the `Run-Transition` marker.
  - Fixed differently, from the self-check:
    - The review proposed that card 12's commit "passes the same trailer" through the trailer commit. I did not do that, because `messageWithTrailers` (`trailercommit.go:99-108`) writes `Warp-SHA` as the local warp HEAD, which need not be the previous transition's tip.
      The give-up instead builds its message with `appendWarpSHATrailer` (the previous transition's `Warp-SHA`) and `appendTrailerLine` (`Run-Transition: <slug>`), and commits through `CommitAnchoredPaths`, which adds no trailer (`commitweftpaths.go:80`).
      **Uses** and `TestReturnToPreviousTransition` changed to match, and the test gained rows for the tip and for skipping fabric commits.
    - The review's `lastTransitionCodeTip` matched the key alone. It now takes the slug and matches the trailer's value, so another run's transition on a shared records branch is not read as this run's.
- **F2 (MEDIUM) — fixed as proposed.**
  Card 6 **Create** gains `SetOwedTipPushedHookForTest`.
  The **Intent** puts it in `export_test.go`, setting the hook and restoring nil in `t.Cleanup` as `SetArchiveTagPushHookForTest` does, and the give-up row names it.
- **F3 (LOW) — fixed as proposed.**
  Card 8 **Edit** gains `internal/loomcli#loomCommitStatusDeps`.
  Its doc's `Spawn` lines are rewritten to name `Push`, `Admit`, `Owed` and `Sleep`, and to say that Commit and Push pass the pre-push check.

`lym loom validate-plan` returns `ok: true`, with only the informational new-unit findings.

## Self-check

- fix-consistency, MEDIUM: card 12's give-up `Warp-SHA` would come from the local HEAD. Fixed with my own change, merged with the matching fix-claims and fix-proposal findings, as described under F1.
- fix-consistency, LOW: `TestReturnToPreviousTransition` had no row for skipping fabric commits. Fixed as proposed.
- fix-consistency, LOW: card 12's single-transition bullet was not updated with the paragraph. Fixed as proposed.
- fix-claims, MEDIUM: same as the first fix-consistency finding. Fixed with my own change, merged.
- fix-claims, LOW: the overview said fabric writes `Warp-SHA` into every weft commit through `commitWeftLocked`. Fixed as proposed: it now names `commitWeftLocked`'s commits and the re-anchor commit separately, without "every".
- fix-claims, LOW: card 6 said other weft commits carry `Warp-SHA` alone. Fixed as proposed: it now says they carry no `Run-Transition`.
- fix-proposal, MEDIUM: same as the first fix-consistency finding. Fixed with my own change, merged.
- fix-proposal, LOW: the `Run-Transition` value was never matched. Fixed as proposed: readers now match the value to the slug.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
