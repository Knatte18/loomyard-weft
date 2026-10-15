## Changes

- F1: added a doc comment above `TestLocateSessions` in `internal/shuttleengine/claudeengine/transcriptread_test.go`, stating that the test is not parallel at the top because one step sets the process-global HOME, and that its table rows run in parallel on temporary roots.
  Fixed as proposed; the rows' `t.Parallel()` was checked.
  Comment-only change, so no test run.

## Disputed

None.

## Deferred

None.
