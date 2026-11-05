## Goal

```markdown
## Seen (operator, 2026-10-31)

The pawl recipe, a beam's run, had a Code-Review perch of its own; the operator did not want it, since the beam's members are already reviewed by their own runs.
The perch is gone: pawl became a loom recipe with no review round (`pawl-loom-recipe`), and Parent-Align brings the parent in before Finalize and Beam-Down land the beam.
Nothing now checks that a beam has reached its project's goal before it lands: Beam-Await ends on a land request.
Bringing the parent into a beam is today done by writing a merge task by hand, as the `codex-engine` beam's two merge-main tasks were.

## Design (operator, 2026-11-01 and 2026-11-03)

- Bringing the parent into a beam is a task of its own: a darn run with its own worktree inside the beam's project, "merge in from parent", with its perch on.
- `lym beam merge-parent <beam>` files such a task in the beam's project with one call, at any time, not only at landing; dobby starts it as any member.
- The run merges the parent into the beam, settles conflicts and gate failures with the merge fixer, and can run Drift.
- Its perch reviews the merge alone; checking the project's goal before the beam lands is no part of it.
- pawl neither starts such a run nor requires one before it lands: pawl lands the beam without a perch round of its own (built by `pawl-loom-recipe`), and its Parent-Align brings the parent in as today.
- Two default buttons in reed's `beam` button group, each with a confirm prompt:
  - `merge-in` runs `lym beam merge-parent {beam}`.
  - `land` runs `lym beam land {beam}`.
- The prime has no parent, so the default `prime` group gets the beam group's other buttons without these two, instead of falling back to the beam group.

Related: note `protected-parent-landing`, landing on a parent the hub may not push to.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-05): The darn writer makes the merge in place of the landing merge fixer

Decision: Accepted departure from the entry's 'settles conflicts and gate failures with the merge fixer': the merge task is a plain darn run whose writer runs lym fabric merge-in and settles conflicts, and the Darn-Write verify gate's cold gate fixer clears gate failures. Review finding F1 is settled by this decision; the code stays as built.

Rationale: The entry's intent is that the run itself settles conflicts and gate failures, which the writer and the cold gate fixer do. The merge fixer belongs to Parent-Align and Finalize, which merge the task's own parent under its lock; pointing them at the beam's parent would need a second parent branch, lock and merge record through landingshed and the darn recipe, for no gain.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
