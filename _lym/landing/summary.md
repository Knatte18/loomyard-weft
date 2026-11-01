# Webster implementer sweeps every retired name across the tree first

The Webster implementer stencil (`contracts/stencils/webster/webster-body-implementer.md`) gains a sweep step for cards that remove or rename a name.
Such a card now retires the old name: a symbol, file, config key or string literal.

In `run-verbs-shared`, the fork for card `06-tapestry-config-module` removed the `loom.plan_*` keys, but its first grep left out `_test` files.
Stale fixtures carrying the old keys then failed across about ten packages, found one gate failure at a time: 69 gate-loop turns of the fork's 165.
Another fork read and edited each file of one grep hit list one at a time over some twenty turns.

## What changes

Before the card's first hand edit, and after any rename tool run, the fork:

1. Greps the whole worktree for every retired name in one Bash call, test files, `testdata/` and other fixtures included.
2. Reads every hit's span in one message.
3. Edits every hit in one message.

For a symbol rename this is the same grep the stencil's rename paragraph already requires for zero references to the old name, widened to the test and fixture files.
It is one grep, not two.

## Design decisions

- The sweep edit is a third sanctioned deviation from the card's targets, next to the guard-step fix and the fix to a failure the card caused.
  It is bounded to hits that reference a name the card retires.
  A hit that matches a retired string by accident, such as a generic literal, is left alone.
- The sweep edit is not a fix to a failure, so the bounded self-fix section's attempt cap does not apply to it.
- The scope guard still records the edits as deviations; they never make a report `FAILED` on their own.

## To check by hand

- The prose is the only change; no code or test was run against it.
  Run a card that retires a name used in a test fixture and confirm the fork sweeps once and the gate finds no stale reference.
- Check that the wording bounding the sweep edit (`sweep-edit-unbounded-failure-fix`, in the added paragraph of step 2 of the stencil) is tight enough:
  a reader should not be able to take it as licence to edit hits unrelated to a retired name, or as exempt from the cap on failure fixes.
  The fix for this finding has not been re-reviewed.
