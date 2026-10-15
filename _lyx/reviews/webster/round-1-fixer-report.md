# Fixer-report, webster round 1

## Changes

- F1, fixed as proposed: `ExternalSender` in `internal/creelengine/seam.go` forms the name with `agentname.FormatProgram(shortname, agentname.External)` and returns `(Sender, error)`.
  `externalFromClaim` and `resolveFromEnvironment` in `internal/creelengine/identity.go` propagate the error to `resolveSender`.
  `internal/creelengine/identity_test.go` and `internal/creelengine/verbs_test.go` use the new signature.
- The refused invalid shortname is an assertion subtest inside `TestResolveSender`, not a table row: the table fixes the shortname at `ly` for every row, and a shortname column would touch each row.
  It covers both the Linux and the non-Linux path.
- `lyx gate test ./internal/creelengine/... ./internal/creelcli/...` passes.

## Disputed

None.

## Deferred

None.
