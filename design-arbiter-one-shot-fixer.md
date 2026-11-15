# The arbiter becomes a one-shot fixer that may edit, not only resume the halted row

`arbiter-one-shot-fixer` · Task · enhancement, arbiter, shed · running · Code-Burler-Fix

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
