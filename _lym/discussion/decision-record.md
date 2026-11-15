## Goal

```markdown
## Seen (operator, 2026-11-15)

The arbiter (`internal/shedarbiter`) handles a stop the Shed driver's rules and tackler leave, but all it may do is a verdict:
resume with a goto held to the halted row, or hand on to the run's parent.
A stop that needs a small edit, to the plan, a record or the code, therefore goes to the orch or the operator, though the arbiter has read enough to make it.

## Idea (operator)

Widen the arbiter in general, for every recipe, into a one-shot fixer: it may edit files to settle the stop, then resume the run.
Weaver (`weaver`) needs it to stand in for Merriam, but the change is the arbiter's, not Weaver's.

## To settle

- What it may edit: the plan and the run's own records, the task's code, or both, and what stays the orch's or the operator's.
- How an edit is committed and checked before the run resumes, so a bad fix is caught by the run's own gates.
- How its edits show in the settle records and the run's history.
```

## Scope

## Decisions

### Added after Discussion (operator, 2026-11-15): Batten's arbiter edits like every other arbiter

Decision: The one-shot fixer applies to every use of the arbiter, batten's included: batten's arbiter gets the same worktree seam and the same edit rights and instructions as loom's, with no batten-specific arbiter instructions.

Rationale: The operator ruled that the widened arbiter is the arbiter everywhere; a verdict-only arbiter in batten would split one role into two behaviours.

### Added after Discussion (parent, 2026-11-15): Batten's arbiter edits in the task worktree, never the prime

Decision: Batten's arbiter seam and the prompt's Worktree name the task worktree (taskWorktreeLocation over the prime and slug), never the hub's prime. At a stop where the task worktree is absent (before it is added or after its teardown), that one call gets no worktree seam and gives a verdict only, and its prompt says so.

Rationale: The prime's HEAD and tree move under other writers, so a read of the arbiter's edits there would claim their commits; the task worktree is the run's own, as it is for loom's arbiter, which keeps one arbiter behaviour everywhere the run has a tree of its own.

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - batten-recipe-and-overview-say-no-fix (class consistency, severity MEDIUM, open in rounds 2)
  - batten-seam-over-prime (class decision, severity MEDIUM, open in rounds 2)
  - template-what-you-may-change-contradicts-fix (class consistency, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
