# Review setup tuning: code_reviewer_fan local, solo profiles, lens changes

Batch 01-code-reviewer-fan-local recorded `done` at head `71d2c26a876e9cc4932de73ec9afa7bef5dd48a2`.
The fork implemented the batch's five plan cards and reported these changes:

- `code_reviewer_fan` moved from the guard category to none in loom's `ConfigSpec`, with its tests and the docs that called it closed.
- `code_reviewer_fan: ""` was set in the `light`, `speed` loom and `docs` profile blocks, with runprofile's doc.
- `board-scope` replaced `goal-scope` and `board-fit`, with the `emphasisLineGaps` test helper.
- `repo-invariants` replaced `way-forward` in `review-plan`, `way-forward` gained the emphasis line, and the `writerFanGaps` helper was added.
- `code-pattern-fit` was folded into `code-affected-set`, and `code-tests` was narrowed.

Deviations: the scope guard found `internal/lenses/config_test.go` changed outside the batch's own targets.
Departures: none.
Renames: none.
