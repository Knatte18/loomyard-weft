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

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
