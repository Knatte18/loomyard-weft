# Fixer-report, webster round 2

## Changes

- F1, fixed as proposed: in `internal/loomcli/smoke_driverstrand_test.go` the three `statusStrandCount` checks now want exactly 1, with reworded messages, and the file comment and test doc comment say an llm-seeded start leaves exactly one status strand beside its driver strand.
- F2, fixed as proposed: in `internal/loomcli/smoke_starttail_test.go` the check wants 1, and the file comment and test doc comment say the start leaves exactly one status strand and no loom-operator strand.

Premise check: `startCmd` calls `ensureStatusStrand` unconditionally after `reed.Up` (`internal/loomcli/start.go:570`).
Both tests carry the `tmux` build tag, so this session could not run them; the change is assertion and comment text only.

## Disputed

None.

## Deferred

None.
