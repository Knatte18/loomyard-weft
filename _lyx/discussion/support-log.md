# Support log — gate-test-runtime-caller-paths

## Interview

A darn run holds no interview: the darn writer read the board entry and its operator decision, and made the calls below itself.

## Rejected alternatives

- Removing `-trimpath` from the tier GOFLAGS: the operator decision keeps it for cache reuse.
- A `go_flags` field in the run record: the log already carries the line, and no reader of the record needs the field.
- Printing the effective GOFLAGS on every tier run, with or without `-trimpath`: the failure mode is `-trimpath` alone, and an unconditional line adds noise to every run.
- Reading only the tier's own `env`: a GOFLAGS inherited from an outer gate run reaches the child too, so the effective environment is what the child sees.

## Review rounds

_No rounds yet._

## Question ledger

- Where does `-trimpath` reach a tier run from? Resolved: the hub's `gate.yaml` tier `env`, not Go code; Go code sets it only for the non-tier holders.
- Should the line fire for an empty scope? Self-pick: no, since no command runs.
