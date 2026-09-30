# Card 3 — Register logger.yaml with configreg

**Edit:**
- `internal/configreg#Modules`
- `internal/configreg/configreg_test.go`
- `internal/configsync/configsync_test.go`

**Uses:**
- `internal/loggerconfig#ConfigTemplate`
- `internal/configsync#ReconcileAll`

**Intent:**
Register `logger` in the config module registry so `lyx config reconcile` recognises `logger.yaml` and seeds it from the template.

Add `{Name: "logger", Template: loggerconfig.ConfigTemplate}` to `Modules()` in alphabetical position, between `landing` and `loom`, as a reconciled (not seed-only) module: its key set is fixed.
In `configreg_test.go`, add `"logger"` to `TestNames`'s `want` list at the same position.
The `configcli` tests that iterate `configreg.Names()` cover the new entry's help text and `--print` section without edits.

In `configsync_test.go`, add a test that `ReconcileAll(dir, true)` over a `t.TempDir()` holding an empty `_lyx/config/` reports a `logger` result with `Applied: true`, and writes `configengine.ConfigFile(dir, "logger")` with content equal to `loggerconfig.ConfigTemplate()`.
This pins the acceptance criterion that reconcile seeds `logger.yaml` from the template.

**ImpactSummary:** Two reach points, no pinned list to edit beyond `TestNames`: every `lyx config` surface that renders the module list (help text, unknown-module error, reconcile output, menu numbering) gains `logger`, and `configsync.ReconcileAll`'s module loop now seeds `logger.yaml`, so every `lyx fabric clone` (`internal/fabriccli/clone.go`) also writes and commits it on the weft primary, as do `hubforge`-built test hubs; the fabric clone commit test derives its module list from `configreg.Modules()`, and the `hubforge` and `configsync` tests read modules by name.

**Commit:** 3: configreg: register logger.yaml
