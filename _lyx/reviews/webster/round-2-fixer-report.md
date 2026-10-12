# Fixer report — webster round 2

## Changes

- F1 (fixed as proposed): `internal/loomcli/smoke_test.go`, the "failure before the first persist" subtest now asserts the driver log contains "invalid state", with the message "want it to name the invalid state".
- F2 (fixed as proposed): `internal/loomcli/smoke_test.go`, the "died driver proceeds to the success envelope and logs why" subtest polls the driver log (25ms sleep, 20s deadline) until it contains "invalid state", then runs the existing assertion on the last content read. `poisonStatusFile` is unchanged.

## Verification

`lyx gate test ./internal/loomcli` passes.
That run covers compilation of the changed file only; the smoke tests themselves are tmux-tier and refused in this session, so neither subtest was executed.

## Disputed

None.

## Deferred

None.
