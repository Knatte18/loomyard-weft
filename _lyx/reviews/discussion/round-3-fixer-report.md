# Round 3 fixer report — discussion

The review's line numbers run ten ahead of the record as read: D4's loom lines are 69 and 71, R3 is 208, and the `LocalDenied` acceptance row is 219.
Both premises hold against the code.

## Changes

- F1 (MEDIUM, consistency), fixed as proposed.
  `internal/loomcli/wiring.go` hands `DiscussionFan` and `PlanFan` to their segments' Bouncer and Burler rows, and `fannedReview` in `internal/loomengine/review.go` returns no fan for an empty value.
  In `decision-record.md` D4, loom's deny list now reads `parent_review_wait_min`, `discussion_fan` and `plan_fan`.
  A new sentence after the `parent_review_wait_min` reason denies the fan keys because each picks the lens set a review segment's forks are given, so a local empty value turns a hub-chosen fanned review back to solo, untracked.
  A second new sentence keeps `discussion_producer` open because it picks the writer, not a reviewer.
  The acceptance row's `LocalDenied` examples add `loom.plan_fan`.
- F2 (LOW, design), fixed as proposed.
  `envsource.Build` overlays `os.Environ()` on `.env` with OS values winning, and `stripTraceID` in `internal/reedengine/lifecycle.go` strips only `LYX_TRACE_ID` from the tmux server's environment.
  R3 now says the process environment overrides `.env`, so a `LYX_*` variable in the environment of the process that boots the hub's shared tmux server reaches every later pane of every pair, and that this cross-pair route belongs to the same parent decision (Q12).
  The closing sentence now covers both routes.
  `support-log.md` Q12 now names the process-environment route through the shared tmux server alongside `.env`.

## Disputed

None.

## Deferred

None.
