# Word the regression check block for the repository reader

The `## Regression check` section that lym appends to a pull request body and landing commit used wording from lym's own run model.
A repository reader saw the baseline described as "the task's branch point" and an off scope as "the task is not regression-checked".
The PR-Review agent cannot edit that section, since lym renders it from the Drift records after the change description.

`internal/driftengine/verdict.go` now renders those lines as:

- `- Baseline: captured at <sha>, the base of this branch.`
- `- Scope: off[, set by <source>]; this change is not regression-checked.`

The Verdict test's rows for the baseline line and the off scope pin the new text.

## Design decisions

- The off-scope line carried the same internal wording in the same block, so it changes with the baseline line.
- The "Amendments by the run's parent" line stays: it names who changed the expectation, which a reviewer needs.
- Internal texts that say "the task's branch point", such as the drift expect stencil, the `drift capture` help and code comments, stay: agents and operators read them, not the repository.

## Merge danger

**Door:** two-way
**Blast radius:** drift
