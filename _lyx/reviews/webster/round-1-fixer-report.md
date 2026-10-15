## Changes

- **F1**: fixed as proposed.
  In the `runTargeting` doc comment in `internal/treadleengine/targeting.go`, dropped "(see the composePrompt convention this mirrors)" so the sentence ends after "still compiles".
- **F2**: fixed as proposed.
  In the `pathRules` comment in `cmd/lyx/pathrules_test.go`, replaced `TestPlanSpec_PatternDirective` with the "PATTERN.md is read from the worktree root" subtest of `TestPlanTable`, which exists at `internal/loomengine/plan_test.go:149`.

Both changes are comment-only, so no test run was needed.
Each fix is its own commit.

## Disputed

None.

## Deferred

None.
