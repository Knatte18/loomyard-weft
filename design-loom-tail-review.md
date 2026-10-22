# Audit: loom from Webster out, who reviews and who fixes, and the target shape

`loom-tail-review` · Note · audit, loom, burler, fabric

## Today: loom from Webster out (`contracts/recipes/loom-recipe.yaml`)

| Row | Reviews | Fixes |
|---|---|---|
| Webster-Review | perch over the code against the plan | Webster fixer (Sonnet) |
| Describe | none; writes the PR text | none |
| Merge-In | none; merges the parent in and runs the verify | `internal/mergeresolve` session resolves conflicts; a verify failure goes back to Webster-Burler-Review |
| Merge-Review | perch over how the task's code meets what the parent gained | merge fixer (Sonnet) |
| Publish | none; landing and publish gates, opens the PR | a gate failure goes back to Webster-Burler-Review |
| PR-Align | alignment agent: scope against the decision record and board, overlap with landed parent commits | none; a reject goes to PR-Rework |
| PR-Gate | records approve (agent or operator), awaits the operator, or a reject | none; a reject goes to PR-Rework |
| PR-Rework | none | writes a new plan generation, then Plan-Review, Webster, Webster-Review and the whole tail again |
| Finalize | none | none; lands: syncs with the parent (moved parent goes back to Merge-In), squashes onto it, marks the board task done, pushes, closes the PR |

## Measured (audit `pr-rework-audit`, landed loom runs 2026-10-08 to 2026-10-22)

- PR-Rework in 8 of 130 runs, never twice; 4 from the alignment agent (of 72 verdicts), all valid; about $4.80 and 23 min each, each new plan one fix card.
- Where those deviations entered: decision record 1, plan 3, code 3 (two written by the Webster-Review or Merge-Review fixer), not real 1.
- Merge-Review ran in 73% of runs, its fixer changed code in a third, about $0.96 per run.
- About 38 verify failures at Merge-In and Publish, sent back to Webster-Burler-Review.

## Problems raised (operator, 2026-10-22)

- Failures at Merge-In and Publish go back to Webster-Review, which is Webster's perch: wrong place.
- A new plan with Plan-Review and the whole Webster pass for a PR-Align reject is overkill.
- A separate Merge-Review perch is not needed.
- Several light fixers in a row can add up to large unreviewed changes, so any review must come after the last fixer.
- The name PR-Align is wrong once the check runs on the branch, before any PR; the Discussion gate's name `parent-review` is wrong too.

## Operator's target shape (operator view, not a decision)

1. Webster.
2. Webster perch.
3. Alignment check, Merge-In and a fixer with gated tests that re-runs them until they pass; maybe one combined fixer.
4. A PR only when a human is to look at it, with its own interactive session that takes the human's feedback and fixes it.
5. With no human in the loop, no PR is needed at all: the alignment check already ran, and no extra review of the PR is needed.

## Operator view, not a decision (2026-10-22, later)

- A PR is only for a human; no LYX-initiated run has had a human reviewing its PR, so without one the run goes straight to Finalize, which already lands locally (squash, push, board done).
- An alignment check against the original scope this late is too late: Discussion and Plan should have caught it.
  Aligning against other board tasks may matter, but is late here too.
- So the only real work after the Webster perch is a thorough Merge-In: conflicts, tests, and fixing after the tests.

Orch judgement: agreed, with two cautions.
- What only exists at the end is the semantic overlap with commits landed since the run started (another run changed the same behaviour or made one of this run's decisions); tests do not catch it.
  The Merge-In fixer's brief can cover it: read what the merge brought in against the decision record, and escalate a conflicting decision instead of fixing it.
- Two of the three code deviations the alignment agent caught were written by a review loop's last fixer, which nothing reviews after it; the fix belongs in the perches, not in a late check.
- Without a PR, Describe's text has no reader unless it becomes the squash commit message.

## Agreed direction (operator, 2026-10-22, for review tomorrow)

- Tail: Webster, Webster perch, Merge-In with its fixer, Finalize.
- Merge-In does conflicts, tests, fixing after tests, and alignment against what the merge brought in.
  Orch caution: it fixes what follows from the decision record, and escalates a real conflict between decisions rather than choosing one.
- The alignment part runs only when the merge brought in commits that touch what the task touches (Merge-Review's `skip_seam: merge-overlap` already detects that); nothing merged in, or nothing overlapping, skips it, since alignment against the task's own scope belongs earlier.
- A separate PR block, an ordinary producer with an entry gate on a config flag: off, the run skips it; on, it publishes the PR, a human is required, and an interactive session takes the human's feedback and fixes it before Finalize.

### A solo fixer at Merge-In (orch recommendation, 2026-10-22)

Yes, with guard rails:
- Conflicts: a solo session already resolves them today (`internal/mergeresolve`).
- Test failures: the gated tests are the fixer's reviewer; it loops until green or its attempts run out, then escalates.
- Alignment with landed commits: it fixes what follows from the decision record and escalates a conflict between decisions.
- A size cap: a fix diff above a threshold, or one reaching outside the files the merge and the failing tests touch, escalates (or gets one perch round) instead of landing unreviewed.
- Its model is a setting.
The time saved is mostly the trips back to Webster-Review after verify failures (about 38 in two weeks), more than the 8 PR-Rework cycles.

## Orch ideas raised in the discussion (not decisions)

- One end-of-tail perch over the diff since Webster-Review approved, skipped when the tail changed nothing; it only works when no fixer runs after it.
- Running every test tier at Merge-In, so Publish has nothing left to fix.
- Names: Discussion-Align and Code-Align for the two alignment points.
- PR-Rework with a new plan kept only for a human reject that changes the goal.

## Open

- What landing without a PR means for Finalize, `alignment_pr_approve`, the PR-closing comment and the GitHub record of a run.
- Whether the combined fixer's changes need a review after it, given point 3 comes after the Webster perch.
- Fixer model and attempt cap before the run escalates.
