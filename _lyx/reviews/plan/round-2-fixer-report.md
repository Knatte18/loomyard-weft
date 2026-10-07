# Round 2 fixer report — plan

All four findings were worked in the plan directory only, with no git commands and the overview's `approved:` key left at `false`.
After the edits `lyx loom validate-plan` reports `ok: true` with informational `create-new-unit` findings only.
No round-1 fix regressed.

## Changes

- **F1.** Card 5's test paragraph now states the rule (every existing test that expects `OutcomeAsking` from `Wait`, `pollEventsTick` or an `Attach` that reaches `Wait` is rewritten to the held reading, ending through the exit it is not about) and lists the tests known today, adding the four the review found:
  `TestRun_Wait_EventsHandling`, `TestRun_Wait_Finalize_OutcomeWriteFailure_StillReturnsClassifiedResult`, `TestPollEventsTick_LaterTurnEndListingExpiredShellEndsWithoutNewWait` and the `BacklogEndsInAsk_ClassifiesAsking` subtest of `TestAttach_OffsetStartsAtZero`.
  `internal/shuttleengine/attach_test.go` joins card 5's Edit targets.
- **F2.** Card 13's driver stencil bullet now names the three sentences of `## Notifying the parent` that change together: the generic line's "and nothing more", the "verbatim" relay of a `parent_notice`, and "add nothing to it".
  Each now admits the tail, so none contradicts it.
- **F3.** The `Clock` rename and `Runner.SetClock` moved out of the corpus card into a new card 14, `14-clock-seam.md`, with a `Create` handle for `SetClock`.
  Its Intent states what the seam lets through (real sleeping and the deadline) and its bound (real-clock defaults, no production caller, a run takes the clock at `Start` or `Attach`, a nil clock is a caller bug).
  Its coverage is `withClock` in `fixture_test.go` applying the clock through `SetClock`, so no new test is added.
  The corpus card is now 15 and the live scenarios card 16; the Card Index, the `notice-shape` `Applies to` line and every in-card cross-reference (cards 12, 14, 15, 16) were updated, and card 15 `Uses` the `SetClock` handle.
- **F4.** Card 5's ImpactSummary no longer pins a package count.

## Deferred

Nothing.
