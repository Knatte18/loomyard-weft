# Fixer report — code round 1

All five findings are fixed, and the gate is green on `./internal/loomrecipe`, `./internal/loomengine` and `./internal/runprofile`.

## Changes

- **F1** (commit `32d4ec7d`): `internal/loomrecipe/arrasrecipe_test.go` gains `TestArrasRun_GotoIntoRaddleWrite` with two subtests.
  - A goto from Raddle-File over a settled perch re-arms it: the second cycle reviews the new proposal and the run ends done.
  - A goto while the perch awaits its parent on a circling escalation (budget 1, hard cap 4) raises the same escalation again.
    It adds no review round and files nothing, and the round-1 review is kept.
  - `newArrasRun` takes an `arrasRunOptions` value (`fileHaltsOnStuck`, `splitEscalates`).
  - **Fixed differently:** the review proposed rows in `TestArrasRun_SplitPerchReviewsEveryProposalBeforeFiling`.
    Its loop runs one `Run` to completion, and these cases need a halt, a goto and a second `Run`, so they live in their own test function.
    The "below the hard cap" case uses a circling escalation, which the soft-cap rule raises from a design-wrong key open in two rounds.
- **F2** (same commit): `TestArrasRecipe` loses `arrasRowNames` with its `slices.Equal` check, the `HoldsSplitPerch(arras)` assertion and the `DiscussionApprovalRow(arras)` assertion.
  The Motif skip, routing, segment, absent-seam and fixer-gate assertions stay.
- **F3** (commit `c8eb0511`): rows added to existing tests.
  - `TestResolveJudge`: a `split` expectation on every row, plus two `split_judge` rows.
  - `TestResolveReview`: the empty-raddle split case (`Split` equals the run-wide models, `SplitKeys` on `fixer_model`), and the raddle-set case (engine `other`, key `raddle`).
  - The `LoadConfig` budget refusal test: negative `split_max_bounces` and `split_hard_max_bounces`.
  - `TestBouncerRunSubdir`: `Raddle-Bouncer` to `split`.
  - `TestTemplateLadder`: each profile's `arras` block holds the discussion and split rounds with a hard cap one above.
  - The raddle-set case in `TestResolveReview` goes one step past the review's list; it is a gap in the same fallback branch.
- **F4** (commit `0f344080`): `docs/overview.md` names `arras` in the batten recipe list and the `Run-Shed` list, with a one-line description of the recipe.
- **F5** (same commit): `docs/overview.md` describes `split_max_bounces`, `split_hard_max_bounces`, `split_auto_accept`, `split_reviewer_model`, `split_fixer_model` (empty takes the `raddle` model), `split_judge`, and the closed `split_reviewer_fan` and `split_fixer_fan`.

F1 and F2 share one commit and F4 and F5 share one, because each pair edits one file.

## Gate repair

The gate flagged the file header of `arrasrecipe_test.go` for a comment wrapped mid-sentence.
The header now breaks at sentence boundaries; no other change.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

Every fork returned a report; the reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

## Disputed

None.

## Deferred

None.

## Rulings

None.
