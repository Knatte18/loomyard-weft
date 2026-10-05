# Card 4 — configreg declares open maps and reconcile carries them

**Edit:**
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configsync#ReconcileAll`
- `internal/configreg/configreg_test.go`
- `internal/configsync/configsync_test.go`

**Create:**
- `plan:internal/configreg#Lookup` -> `func Lookup(name string) (Module, bool)`

**Uses:**
- `plan:internal/boardengine#ConfigOpenMaps`
- `internal/yamlengine#Reconcile`

**Intent:**
`Module` gains an `OpenMaps []string` field next to `Template`, and the `board` entry sets it from `boardengine.ConfigOpenMaps()`.
`Lookup` returns the whole registered `Module` for a name, so a caller gets the template and its open maps from one place.
`ReconcileAll` passes `m.OpenMaps` to `yamlengine.Reconcile`, so `lyx config reconcile` neither reports nor removes repo-specific labels and adds none of the template's (D3).
No import cycle: `configreg` already imports `boardengine`, and the engines import neither `configreg` nor `configsync` (R3).

Tests: `Lookup("board")` returns `types` and `labels` as open maps and `Lookup("bogus")` reports false.
`ReconcileAll` on a `board.yaml` whose `types` and `labels` are maps of non-template keys reports no added or removed key and, with apply, writes them unchanged; a list-shaped `labels` is carried whole.

**ImpactSummary:** the registry struct gains a field and a lookup; reconcile changes only for modules that declare open maps.

**Commit:** 4: configreg declares open maps and reconcile carries them
