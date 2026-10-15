# Fixer report — plan round 1

## Changes

All in `_lyx/plan/04-loom-gate-and-verb.md`.

- **F1** (fixed as proposed): added `internal/loomcli#TestWire_DefaultConfig` and `internal/loomcli#TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig` to the Edit list after `internal/loomcli#loomCLI.wireLightweight`.
  Added a Tests bullet: each gains one assertion, beside its existing `SupportLogPath` assertion (`internal/loomcli/wiring_test.go:208`, `:729`), that `c.env.RelevantPath` equals `loomengine.DiscussionRelevant(location)` (`loc` in `TestWire_DefaultConfig`).
- **F2** (fixed as proposed): `IOFault_RelevantIsDirectory` now wants the message to contain `loom: validate discussion: `, which the old `loom: validate discussion record <path>: ` prefix does not contain; the bullet says the relevant path comes from the wrapped read error.
- **F3** (fixed as proposed): "gains two rows" is now "gains three rows".

`lyx loom validate-plan` returns ok after the edits.

## Disputed

None.

## Deferred

None.
