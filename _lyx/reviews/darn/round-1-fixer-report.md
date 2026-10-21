# Fixer-report, darn round 1

## Changes

- F1 (MEDIUM): fixed as proposed.
  `internal/configengine/config_test.go`: `loadCase` gained a `perNameMaps` field passed into the spec, and `TestLoad` gained two rows:
  a name missing under a declared per-name map is filled and not logged, and a missing per-name map path is still logged.
- F2 (LOW): fixed as proposed.
  The `PerNameMaps` comment in `internal/configreg/configreg.go` now says configsync keeps only the names the file changed, leaves out a name holding the template's value, never copies a template name in, and keeps every entry the template lacks.
  The gate found that comment hard-wrapped mid-sentence; it is now one sentence on one line.
- F3 (LOW): fixed as proposed.
  The `lyx config reconcile` Long text in `internal/configcli/configcli.go` names `dropped` and says a file is rewritten when the reconcile adds, removes or drops anything.
  The self-check's affected-set fork found the same wording stale in `internal/configcli/doc.go`, so the module doc and the "adds and removes nothing" clause of the help text were updated too.

`lyx gate test` passes for `./internal/configengine`, `./internal/configreg` and `./internal/configcli`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: LOW, `internal/configcli/doc.go` reconcile paragraph still said apply writes when the reconcile "adds and removes nothing" and never named `dropped`; fixed as proposed, with the help text's matching clause.

## Disputed

None.

## Deferred

None.
