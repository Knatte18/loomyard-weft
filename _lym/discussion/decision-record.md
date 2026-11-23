## Goal

```markdown
Imported from [issue #192](https://github.com/Knatte18/loomyard/issues/192).

Seen in a C# hub (NORCE Models), darn run hydraulics-gaps: the red-first gate held the writer's handoff because it cannot run the diff's tests on the merge base when the hub's scoper is not go. The run went on only after the parent orch recorded a decision titled 'No test seam ...' by lym loom decision add, waiving the check. Workaround: the parent records a 'No test seam' decision naming why the check cannot run. Expected: in a hub whose scoper cannot run tests on the merge base, the gate skips red-first or runs it through the hub's own test command, instead of halting every run for a manual waiver.
```

## Scope

## Decisions

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - darn-recipe-red-first-comment-stale (class consistency, severity MEDIUM, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
