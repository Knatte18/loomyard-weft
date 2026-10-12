# Fixer report — round 1

## Changes

- F1: added the row `invalid JSON is malformed` to `TestValidatePlanCmd_Overlay` in `internal/loomcli/overlay_test.go`.
  The input is a truncated object (`{"a":`), expecting exit 1 and `malformed overlay` plus `way forward: pipe one JSON object`.
  It is a row, so `test-budget.yaml` is unchanged.
  `lyx gate test ./internal/loomcli` is green.

## Disputed

None.

## Deferred

None.
