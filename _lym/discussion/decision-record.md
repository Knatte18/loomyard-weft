## Goal

```markdown
Imported from [issue #128](https://github.com/Knatte18/loomyard/issues/128).

/home/AD.NORCERESEARCH.NO/hanf/Code/NORCE-DrillingAndWells/DrillOpPlan-LYMHUB/adopt-models-sim/.scratch/selfreport-merge-parent-slug.md

## Issue comment, 2026-11-13

Operator's expectation: a run's directory `_lym/shed/<slug>/` should be cleaned up when the run finishes (after Land and Worktree-Teardown). Here the first merge-in's run directory from 2026-11-10 was still in the prime with state `done`, which is what made batten refuse the slug. So the fix is that teardown removes the finished run's directory (and its task branch), not only that merge-parent picks a fresh slug.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - docs-describe-old-done-run-refusal (class consistency, severity MEDIUM, open in rounds 1)
  - done-run-clear-leaves-task-branch (class scope, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
