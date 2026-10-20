# Card 2 — configengine per-name spec

**Edit:**
- `internal/configengine#Spec`
- `internal/configengine#warnUnknownKeys`
- `internal/configengine#Set`
- `internal/configengine#setSparse`
- `internal/configengine/doc.go`
- `internal/configengine_test#TestLoad_LayerStack`
- `internal/configengine_test#TestSet`

**Intent:** `configengine.Spec` gains a `PerNameMaps []string` field, the dotted paths of the module's per-name maps (card 1's kind), documented in the `Spec` comment beside `OpenMaps`.
Every template comparison a spec drives honours it, per D2:
`warnUnknownKeys` passes `spec.PerNameMaps` to `yamlengine.Prune`, so a name under a per-name map that the template lacks is never warned as a key no loader reads;
`Set` passes it to `yamlengine.SetValues` and `setSparse` to `yamlengine.SetSparse`, so `--set` knows `P.<name>` for any name on the base and the sparse layers.
The load's template fill needs no change: `yamlengine.FillMissing` already appends a template name the stack lacks under a mapping, so a missing template lens or fan resolves to its template value.
The local layer is untouched: `ValidateLocal` still refuses what it refuses, and a guard-classified per-name path stays closed to it.
The package doc gains a per-name maps paragraph beside "# Open maps", saying a `Spec`'s `PerNameMaps` are passed unchanged to yamlengine by the unknown-keys warning and by `Set` on the base and sparse layers.

Rows: `TestLoad_LayerStack` gains a `perNameMaps` field on its spec and a row whose default-layer file holds a name under a per-name map that the template lacks, asserting no unknown-keys warning over both loaders, while a key outside the map in the same file still warns;
`TestSet` gains a `perNameMaps` field and two rows, one setting a name the template lacks on the base layer and one on a sparse layer (`board: true`), each asserting the name is written.
The unknown-key refusal under a per-name map is card 1's `TestSetValues_UnknownKeys` row; `Set` only hands the paths through.

**ImpactSummary:** One new Spec field read by three configengine members; every registered module leaves it empty until card 3, so no module's load, warning or set changes yet.
