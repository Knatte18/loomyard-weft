# Word the regression check block for the repository, not the task

The `## Regression check` block that lym appends to a pull request and landing commit used internal process wording.
Its baseline line read "the task's branch point", and its scope-off line read "the task is not regression-checked".
A reader of the repository has no notion of a task, and the PR-Review agent cannot edit the appended block.

`internal/driftengine/verdict.go` now renders:

- the baseline line as "Baseline: captured at `<sha>`, the base of this branch.";
- the scope-off line as "this change is not regression-checked", with and without a named scope source.

`verdict_test.go` pins the new wording.
Only text changes: no record, key or format a reader parses moves.

Closes #204

## Merge danger

**Door:** two-way
**Blast radius:** driftengine
