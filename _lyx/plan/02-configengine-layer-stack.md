# Card 2 — configengine layer stack

**Create:**
- `plan:internal/configengine#Dirs` -> `type Dirs struct`
- `plan:internal/configengine#Layer` -> `type Layer string`
- `plan:internal/configengine#LayerTemplate` -> `const LayerTemplate Layer = "template"`
- `plan:internal/configengine#LayerDefault` -> `const LayerDefault Layer = "default"`
- `plan:internal/configengine#LayerWorktree` -> `const LayerWorktree Layer = "worktree"`
- `plan:internal/configengine#LayerLocal` -> `const LayerLocal Layer = "local"`
- `plan:internal/configengine#Category` -> `type Category string`
- `plan:internal/configengine#CategoryNone` -> `const CategoryNone Category = ""`
- `plan:internal/configengine#CategoryExecutable` -> `const CategoryExecutable Category = "executable"`
- `plan:internal/configengine#CategorySourcedFile` -> `const CategorySourcedFile Category = "sourced-file"`
- `plan:internal/configengine#CategoryCrossPair` -> `const CategoryCrossPair Category = "cross-pair"`
- `plan:internal/configengine#CategoryGuard` -> `const CategoryGuard Category = "guard"`
- `plan:internal/configengine#Spec` -> `type Spec struct`
- `plan:internal/configengine#LocalFile` -> `func LocalFile(anchor, module string) string`
- `plan:internal/configengine#LayerFile` -> `func LayerFile(dirs Dirs, spec Spec, layer Layer) (string, bool)`
- `plan:internal/configengine#BaseLayer` -> `func BaseLayer(dirs Dirs, spec Spec) Layer`
- `plan:internal/configengine#LocalKeys` -> `func LocalKeys(spec Spec) []string`
- `plan:internal/configengine#LocalDenied` -> `func LocalDenied(spec Spec) []string`
- `plan:internal/configengine#ValidateLocal` -> `func ValidateLocal(spec Spec, doc []byte) error`
- `plan:internal/configengine#LayerDoc` -> `type LayerDoc struct`
- `plan:internal/configengine#Layers` -> `func Layers(dirs Dirs, spec Spec) ([]LayerDoc, error)`
- `plan:internal/configengine_test#TestLoad_LayerStack` -> `func TestLoad_LayerStack(t *testing.T)`
- `plan:internal/configengine_test#TestValidateLocal` -> `func TestValidateLocal(t *testing.T)`
- `plan:internal/configengine_test#TestLayers` -> `func TestLayers(t *testing.T)`

**Edit:**
- `internal/configengine#Load` -> `func Load(dirs Dirs, spec Spec) ([]byte, error)`
- `internal/configengine#LoadOrTemplate` -> `func LoadOrTemplate(dirs Dirs, spec Spec) ([]byte, error)`
- `internal/configengine#load` -> `func load(dirs Dirs, spec Spec, fallbackOnAbsent bool) ([]byte, error)`
- `internal/configengine#fallbackToTemplate`
- `internal/configengine_test#TestLoad`
- `internal/configengine_test#TestLoad_Refusals`
- `internal/configengine_test#TestLoadOrTemplate`
- `internal/configengine_test#TestLoadOrTemplate_UnsetRequiredEnv_WrapsAsConfigTemplate`
- `internal/configengine_test#TestLoadOrTemplate_LyxDirStatFailure_DoesNotFallback`
- `internal/configreg_test#loadFilled`
- `internal/reedengine#LoadConfig`
- `internal/shuttleengine#LoadConfig`
- `internal/orchengine#LoadConfig`
- `internal/websterengine#LoadConfig`
- `internal/loggerconfig#Load`
- `internal/batcher#loadConfig`
- `internal/loomengine#LoadConfig`
- `internal/darnengine#LoadConfig`
- `internal/landingshed#LoadConfig`
- `internal/gateslot#LoadConfig`
- `internal/boardengine#LoadConfig`
- `internal/fabricengine#LoadConfig`
- `internal/configcli#validateHubFile`
- `internal/configengine/doc.go`
- `pattern/PATTERN-config-strictness.md`
- `pattern/PATTERN-durable-vs-ephemeral-state.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/yamlengine#Merge`
- `plan:internal/yamlengine#LeafPaths`

**Intent:**
Give configengine the layer stack of D1, D2, D3 and D5, behind the two loaders whose names and strictness split stay as `PATTERN-config-strictness` pins them.
The overview's Shared Decisions "Layer geometry" and "Local-layer rule" are this card's contract.

New types, in `config.go` or a new file beside it:

- `Dirs` holds `Anchor`, the reading worktree's anchor, and `Board`, the hub's board dir, empty standalone; both are told, never derived (`PATTERN-told-geometry`).
- `Layer` and its four constants name a layer for provenance, `--layer` and reconcile results.
- `Category` and its constants are D4's categories; `CategoryNone` meets none.
- `Spec` describes one module: `Name`, `Template []byte`, `OpenMaps []string`, `HubWide bool`, `WholeEntries bool` (each top-level key is one unit, open to the local layer: `models`'s aliases), and `Categories map[string]Category`, the module's per-key classification, keyed by dotted path, an entry covering its own path and every path under it.

New functions:

- `LocalFile` returns `<anchor>/.lyx/config/<module>.local.yaml`, in `ScratchDir`'s directory and never at `StagingFile`'s path.
- `LayerFile` returns a layer's file and whether that layer applies: `LayerDefault` is `ConfigFile(Board, name)` when `Board` is set; `LayerWorktree` is `ConfigFile(Anchor, name)` when `Anchor` is set and the module is not hub-wide or `Board` is empty; `LayerLocal` is `LocalFile(Anchor, name)` when `Anchor` is set; `LayerTemplate` never has a file.
- `BaseLayer` is `LayerDefault` when `Board` is set, else `LayerWorktree`.
- `LocalKeys` returns the sorted `Categories` paths classified `CategoryNone`, `LocalDenied` the sorted paths classified otherwise.
- `ValidateLocal` checks a local document under D5: every unit path (`yamlengine.LeafPaths` with the spec's open maps and `WholeEntries`) must exist in the template, sit under an open map or, with `WholeEntries`, under any top-level key; its shape (mapping or not) must match the template's where the template holds the path; and the longest `Categories` entry covering it must be `CategoryNone`, while a hub-wide module also refuses a path no entry covers.
  A failure is marked `ErrInvalid` and names the key; an empty or comment-only document passes.
- `LayerDoc` holds one present layer file: its `Layer`, `Path` and raw `Bytes`.
  `Layers` returns the present file layers in stack order, base first, each parsed and the local one checked by `ValidateLocal`, a failure marked `ErrInvalid` naming the file; an absent file is skipped and an empty or comment-only local file is dropped.
  It is the one reader of layer files: the loaders stack its result, and the seed-only readers (`models`, `burler`), which validate per file and apply their own fallback, read through it.

`Load` and `LoadOrTemplate` keep their split, now over the base layer: the `_lyx/` probe runs on `Anchor` when set, else on `Board`; strict `Load` errors on an absent base file, naming it, with today's `lyx config reconcile` way forward for a worktree base and, for a base under the board dir, which only the hub-wide reconcile seeds, `run "lyx fabric reconcile"` followed by the clause `; a session lyx refuses the verb from reports status: FAILED and the orch runs it`; and `LoadOrTemplate` degrades on an absent `_lyx/` or base file by taking the template as the base, still stacking the upper layers present.
An absent upper file adds nothing, and so does an empty or comment-only local file.
A present layer file that does not parse returns `ErrInvalid` naming it, and a local file failing `ValidateLocal` returns `ErrInvalid` naming the file and the key.
The base layer and the worktree file keep today's tolerance of keys the template lacks.
The layers merge unexpanded in stack order, then `FillMissing` fills from the template and logs the filled keys with the layer `template`, `MissingKeys` runs as today, and the merged document is expanded once through `envsource.Build` of `Board` for a hub-wide module and of `Anchor` otherwise (`Board` when `Anchor` is empty).
`fallbackToTemplate` folds into that path or stays as its tail, the implementer's call.

Every caller moves to the new signature with a transitional geometry that keeps today's behaviour exactly, until its module's own card re-signs its loader: a per-worktree module's loader passes `configengine.Dirs{Anchor: baseDir}` and a hub-wide module's `configengine.Dirs{Board: baseDir}` (its `baseDir` is the board dir today), each with a `configengine.Spec` literal carrying `Name`, `Template`, `OpenMaps` and, for darn, landing, gate, board and fabric, `HubWide: true`.
`validateHubFile` passes `Dirs{Board: boardDir}` and the module's flags; `loadFilled` passes `Dirs{Anchor: dir}`.

Tests at the package surface: the existing loader tests take the new signature unchanged in what they assert, except that `TestLoad_Refusals` gains the absent board-dir base row naming `lyx fabric reconcile` beside its absent worktree-base row naming `lyx config reconcile`; `TestLoad_LayerStack` is table-driven over a temp board dir and anchor and pins every resolution row of the acceptance criteria that configengine owns: template fallback at every depth, highest layer wins, list and scalar whole replacement, open map whole from the highest layer, standalone stack of template, worktree and local, strict absent base and degrading absent base with an upper layer still applied, absent upper and empty or comment-only local as no error, a stale `StagingFile` never read as the local layer, and a `${env:...}` marker the worktree file inherits from the default expanding against the anchor's `.env`; `TestValidateLocal` pins the unknown key, the shape mismatch, an open-map entry and a `WholeEntries` alias accepted, a hub-wide uncovered key and a non-none category refused, each naming the key; `TestLayers` pins the stack order per geometry (hub per-worktree, hub hub-wide, standalone), skipped absent files, a dropped comment-only local file and a local refusal naming the file.

`doc.go` describes the stack, the file locations, strictness per base layer, the local rule, single post-merge expansion, and that local override files are never disposable for any cleanup.
`pattern/PATTERN-config-strictness.md` states that strictness applies to the base layer, the fill runs down to the template, and a local file refuses unknown keys; `pattern/PATTERN-durable-vs-ephemeral-state.md` states that the local override sits in the mirrored directory, not at the mirrored file path, which is the editor staging copy's.

**ImpactSummary:** re-signs both loaders: twelve module loaders, `loggerconfig.Load`, configcli's hub validation and two test helpers move to a behaviour-preserving transitional geometry.
