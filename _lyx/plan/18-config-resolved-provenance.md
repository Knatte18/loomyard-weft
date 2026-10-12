# Card 18 — config resolved provenance

**Create:**
- `plan:internal/configengine#Provenance` -> `func Provenance(dirs Dirs, spec Spec) ([]byte, error)`
- `plan:internal/configengine_test#TestProvenance` -> `func TestProvenance(t *testing.T)`
- `plan:internal/configcli#printResolved` -> `func printResolved(dirs configDirs, out io.Writer, module string) int`
- `plan:internal/configcli#runResolved` -> `func runResolved(ctx context.Context, out io.Writer, module string) int`
- `plan:internal/configcli#TestPrintResolved` -> `func TestPrintResolved(t *testing.T)`

**Edit:**
- `internal/configcli#Command`
- `internal/configcli#buildConfigLong`
- `internal/configcli/doc.go`
- `internal/configengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/configengine#Layers`
- `plan:internal/yamlengine#Merge`
- `plan:internal/yamlengine#LeafPaths`
- `plan:internal/yamlengine#AnnotateLeaves`
- `plan:internal/configcli#configDirs.layerFile`

**Intent:**
Add D7's `--resolved`: `lyx config <module> --resolved` prints the merged result each leaf annotated with the layer it came from, `template`, `default`, `worktree` or `local`.
`configengine.Provenance` stacks the module's layers exactly as `Load` does, through `Layers`, the merge, the template fill and the single expansion against the same dir, and names each unit the highest layer whose document holds it, or `template` when none does, with `yamlengine.AnnotateLeaves` writing the name as the unit's line comment; it applies `Load`'s strictness and refusals unchanged, never degrading, so an absent base layer errors as `Load` does.
`Command` gains the `--resolved` flag, mutually exclusive with `--print`, `--set` and `--layer`, and requiring a module; its RunE routes it to `runResolved`, which resolves the location as `runConfig` does and calls `printResolved`, which looks the module up, builds the dirs from `configDirs` and writes `Provenance`'s bytes, or an `output.Err` envelope naming the failure.
`buildConfigLong`, configcli's `doc.go` and configengine's `doc.go` document it.

`TestProvenance` is table-driven over a temp board dir and anchor: a key only the template holds is marked `template`, one the default sets `default`, a worktree override `worktree`, a local override `local`, an open map marked by the highest layer holding it, and a refused local key returns `ErrInvalid` naming the file.
`TestPrintResolved` drives the CLI path over a hub fixture, asserting the annotated output for loom and the refusal of `--resolved` beside `--set`.
The budget rows for `internal/configengine` and `internal/configcli` rise by one each.

**ImpactSummary:** additive: one configengine function and a read-only flag on `lyx config`; no existing path changes.
