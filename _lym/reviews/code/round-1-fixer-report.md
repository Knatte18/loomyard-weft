# Round 1 fixer-report

## Changes

Commit `9e5a0d36` changes `contracts/stencils/loom/loom-rubric-raddle-review.md`; both findings sit in that file and one commit holds both.

- **F1, fixed as proposed.**
  The `design` class sentence now names "a part added only so parts run at once" and "a cut that runs fewer parts at once than another cut with the same part count" in place of the "part count allows" case.
  The lines are split at the commas to stay under the line limit.
- **F2, fixed as proposed.**
  The judgement bullet now reads "the one with a short `depends_on` chain and the most parts with no `depends_on` path between them", and its finding sentence names a cut that runs fewer parts at once than another cut with the same part count.
  The first half's wording joins the sentence's "the one with" lead-in, so it reads as one clause.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW, packages-shared half dropped from the rubric's count: disputed, `premise false`.
- fix-proposal, LOW, rubric finding condition counts only parallelism while the judgement names chain too: disputed, `every fix worse`.

## Disputed

- **fix-proposal LOW 1, premise false.**
  The reviewer already placed the different-packages half in the `raddle` gate's `packages-shared` check (`internal/raddleparser/check.go:398`), and the rubric's "Already enforced" section says the gate passes every proposal before review.
  The lens itself reports the case as moot.
- **fix-proposal LOW 2, every fix worse.**
  A condition that also flags a longer chain must rank the two measures, and the record (`_lym/discussion/decision-record.md:22`) names both without ranking them.
  Any wording that flags the longer-chain cut flags both cuts of a pair where one has the shorter chain and the other more parallel parts.
  The reviewer's proposal, which the fix carries out, keeps the finding to a cut another cut beats on parallelism at equal part count.

## Deferred

Nothing deferred.

## Rulings

No parent ruling reached me.
