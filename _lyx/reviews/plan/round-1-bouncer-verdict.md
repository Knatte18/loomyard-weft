---
verdict: APPROVED
rationale: "All five round-1 findings (F1-F5) are fixed in cards 2 and 4 as the report asked; D1-D10 and every Constraint are carried; verify runs shedengine, battenshed and battencli including -tags integration; no new finding survived the Do-not-flag list"
---

## Round-1 findings, checked against the current cards

- **F1 (MEDIUM), fixed.**
  `02-step-fold-wiring.md` now has a `step_fold_test.go` case where a pause write lands right after a fold.
  The case asserts that history is unchanged and `activity.last` is carried forward, and that the next step-3b resume write carries it again while the exempt Stuck folds across the pause (D4) to `Repeats` 2.
  I checked the discriminator against `internal/shedengine/run.go`.
  The step-3 pause write, the step-3b resume write and the cancellation arm all pass `st.History` unchanged.
  So does the producer-error arm with an empty outcome, since `appendHistory` returns an unchanged copy.
  Every fold increments the last entry's `Repeats`, so "longer, or equal non-empty length with a differing last `Repeats`" separates a verdict write from a no-verdict write.
- **F2 (LOW), fixed.**
  The Awaiting-interleave case now scripts exempt, exempt, Awaiting, exempt, which matches the acceptance criterion.
  It also says the step after the halt is a plain `Step`.
  That matches `stepLocked`, which does not short-circuit on `StateAwaiting` and re-calls the producer through step 3b.
- **F3 (LOW), fixed.**
  Card 2's ImpactSummary is now a real conclusion.
  Only battenshed emits exempt Stucks, so batten's `history_length` freezes across a wait (D10) and battencli's 20-entry window spans more of it, while loomengine's readers are unaffected.
- **F4 (LOW), fixed.**
  `internal/battencli#maxStatusHistoryEntries` is an `Edit:` target, and the Intent rewords its stale one-entry-per-poll comment.
  The constant and `recentHistory` stay unchanged, so battencli's truncation behavior, which the decision record puts out of scope, is untouched.
  I ratify this as within scope: the comment makes the same claim `docs/overview.md` corrects, and the change is a doc correction with no behavior change.
- **F5 (NIT), fixed.**
  Card 4's third-Call bullet now states that the child's history length equals the recorded one, so it is the D9 regardless-of-length case.

## Independent checks

- **Fidelity.**
  D1–D5 are carried by card 1, D2 and D6 by card 2, D7–D9 by cards 3 and 4, and D10 by card 2's overview text and ImpactSummary.
  The single-persist Constraint is covered twice: card 1 requires a fresh slice and tests that the input is unchanged, and card 2 tests that one `CommitStatus` call carries the fold.
  The other Constraints are covered as follows: `ReadJSONStrict` by fields declared on `HistoryEntry` (card 1), Told-Geometry by the marker path staying `approvalActedFile` (card 4), and docs in the same commit by card 2.
  Card 3 reads an unparseable second line as the old layout, which follows D8's conservative rationale.
  The review ratified that reading, and I agree it adds no scope.
- **Existing tests.**
  `budgetexempt_test.go`'s interleave and budget tests assert state and reason, not history length, so folding does not break them.
  No battenshed test asserts one entry per exempt poll.
- **Verify coverage.**
  Every targeted package is run with and without `-tags integration`.
  These three packages carry no other build tags.
- **Granularity.**
  Card 3 holds pure helpers with their own table tests, so it is an independently testable unit and not a split for its own sake.
- **Typing.**
  There are no `Custom` cards.
  Both `Edit` cards carry a concrete ImpactSummary.

The round-1 report has no `## Focus departures` section, so there is nothing to ratify or reject there.
