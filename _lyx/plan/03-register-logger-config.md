# Card 3 — Register logger.yaml with configreg

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg/configreg_test.go`

**Uses:**
- `plan:internal/loggerconfig#ConfigTemplate`

**Intent:**
Register `logger` in the config module registry so `lyx config reconcile` recognises `logger.yaml` and seeds it from the template.

Add `{Name: "logger", Template: loggerconfig.ConfigTemplate}` to `Modules()` in alphabetical position, between `landing` and `loom`, as a reconciled (not seed-only) module: its key set is fixed.
In `configreg_test.go`, add `"logger"` to `TestNames`'s `want` list at the same position.
The `configcli` tests that iterate `configreg.Names()` cover the new entry without edits.

**ImpactSummary:** Every `lyx config` surface that renders the module list (help text, unknown-module error, reconcile output, menu numbering) gains `logger`.

**Commit:** 3: configreg: register logger.yaml
