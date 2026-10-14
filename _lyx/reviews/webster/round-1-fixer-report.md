## Changes

- F1: `statusCmd` in `internal/tendercli/cli.go` now names the live-session listing and the replaced-binary processes in `Short` and `Long`. Fixed as proposed.
- F2: `docs/overview.md` tender bullet names `revive` in the caller rule. Fixed as proposed.
- F3: `Command` doc comment names the revive verb and `RunCLI` doc comment says "tender module CLI". Fixed as proposed.
- F4: `docs/overview.md` shortcuts bullet has the comma before `lyx prime`. Fixed as proposed.

Each fix is its own commit. `lyx gate test ./internal/tendercli` passes.

## Disputed

None.

## Deferred

None.
