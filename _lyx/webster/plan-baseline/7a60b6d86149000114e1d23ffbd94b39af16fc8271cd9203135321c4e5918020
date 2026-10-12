# Card 2 — configengine layer stack

**Create:**
- `internal/configengine#Dirs`
- `internal/configengine#Layer`
- `internal/configengine#LayerTemplate`
- `internal/configengine#LayerDefault`
- `internal/configengine#LayerWorktree`
- `internal/configengine#LayerLocal`
- `internal/configengine#Category`
- `internal/configengine#CategoryNone`
- `internal/configengine#CategoryExecutable`
- `internal/configengine#CategorySourcedFile`
- `internal/configengine#CategoryCrossPair`
- `internal/configengine#CategoryGuard`
- `internal/configengine#Spec`
- `internal/configengine#LocalFile`
- `internal/configengine#LayerFile`
- `internal/configengine#BaseLayer`
- `internal/configengine#LocalKeys`
- `internal/configengine#LocalDenied`
- `internal/configengine#ValidateLocal`
- `internal/configengine#LayerDoc`
- `internal/configengine#Layers`
- `internal/configengine#Stack`
- `internal/configengine#StackLayers`
- `internal/configengine_test#TestLoad_LayerStack`
- `internal/configengine_test#TestValidateLocal`
- `internal/configengine_test#TestLayers`

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
- `internal/yamlengine#Merge`
- `internal/yamlengine#LeafPaths`

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
  It is the one reader of layer files.
- `Stack` holds one module's stacked layers: `Doc`, the present file layers merged unexpanded in stack order through `yamlengine.Merge` under the spec's open maps and `WholeEntries`, with no template fill; `Layers`, the `Layers` result it was merged from; and `Origins`, mapping each top-level key `Doc` holds to the highest `LayerDoc` holding it.
  `StackLayers` returns it, with `Layers`'s errors; no present layer yields an empty `Doc`.
  It is the one place layers are merged: `Load` and `LoadOrTemplate` merge through it, a seed-only reader (`models`, `burler`) validates each of its `Layers` naming that file and decodes its `Doc`, and a reader naming the file a top-level key came from reads `Origins` (`PATTERN-config-layers`).

`Load` and `LoadOrTemplate` keep their split, now over the base layer: the `_lyx/` probe runs on the base layer's directory, `Board` when set, else `Anchor`; an absent `_lyx/` under `Anchor` while `Board` is set means the worktree and local layers are absent, never an uninitialised base; strict `Load` errors on an absent base file, naming it, with today's `lyx config reconcile` way forward for a worktree base; for a hub-wide module's hub file, which `lyx fabric reconcile` seeds, `run "lyx fabric reconcile"`; and for a per-worktree module's repo default, which only `hubreconcile.Ensure`'s walk seeds and which card 15 makes that walk treat as a stale stamp when absent, `re-run the verb that starts the run, as "lyx loom resume"; its hub reconcile seeds the default from the prime`; each board-dir way forward is followed by the clause `; a session lyx refuses the verb from reports status: FAILED and the orch runs it`; and `LoadOrTemplate` degrades on an absent base-layer `_lyx/` or base file by taking the template as the base, still stacking the upper layers present.
An absent upper file adds nothing, and so does an empty or comment-only local file.
A present layer file that does not parse returns `ErrInvalid` naming it, and a local file failing `ValidateLocal` returns `ErrInvalid` naming the file and the key.
The base layer and the worktree file keep today's tolerance of keys the template lacks.
The layers merge unexpanded through `StackLayers`, then `FillMissing` fills from the template and logs the filled keys with the layer `template`, `MissingKeys` runs as today, and the merged document is expanded once through `envsource.Build` of `Board` for a hub-wide module and of `Anchor` otherwise (`Board` when `Anchor` is empty).
Only template fills are logged: a key a sparse upper layer inherits from a file layer below it is the normal case in a hub, so logging it would log most keys on every load, and `lyx config <module> --resolved` (card 18) names the layer each key came from.
`fallbackToTemplate` folds into that path or stays as its tail, the implementer's call.

Every caller moves to the new signature with a transitional geometry that keeps today's behaviour exactly, until its module's own card re-signs its loader: a per-worktree module's loader passes `configengine.Dirs{Anchor: baseDir}` and a hub-wide module's `configengine.Dirs{Board: baseDir}` (its `baseDir` is the board dir today), each with a `configengine.Spec` literal carrying `Name`, `Template`, `OpenMaps` and, for darn, landing, gate, board and fabric, `HubWide: true`.
`validateHubFile` passes `Dirs{Board: boardDir}` and the module's flags; `loadFilled` passes `Dirs{Anchor: dir}`.

Tests at the package surface: the existing loader tests take the new signature unchanged in what they assert, except that `TestLoad_Refusals` gains two absent board-dir base rows beside its absent worktree-base row naming `lyx config reconcile`: a hub-wide spec's naming `lyx fabric reconcile`, and a per-worktree spec's naming the start verb and the hub reconcile; `TestLoad_LayerStack` is table-driven over a temp board dir and anchor and pins every resolution row of the acceptance criteria that configengine owns: template fallback at every depth, highest layer wins, list and scalar whole replacement, open map whole from the highest layer, standalone stack of template, worktree and local, strict absent base and degrading absent base with an upper layer still applied, absent upper and empty or comment-only local as no error, a stale `StagingFile` never read as the local layer, a `${env:...}` marker the worktree file inherits from the default expanding against the anchor's `.env`, a template fill logged with the layer `template` while a key the worktree file inherits from the default logs nothing, and two rows with `Dirs{Anchor: <dir without _lyx/>, Board: <initialised board dir>}`: a hub-wide spec loads its hub file under strict `Load`, and a per-worktree spec resolves a repo-default key under both `Load` and `LoadOrTemplate`; `TestValidateLocal` pins the unknown key, the shape mismatch, an open-map entry and a `WholeEntries` alias accepted, a hub-wide uncovered key and a non-none category refused, each naming the key; `TestLayers` pins the stack order per geometry (hub per-worktree, hub hub-wide, standalone), skipped absent files, a dropped comment-only local file and a local refusal naming the file, and `StackLayers` rows: a `Doc` merged with no template key filled, a mapping merged per entry while a scalar and a list are replaced whole, a `WholeEntries` entry replaced whole, an `Origins` entry naming the highest layer holding each top-level key, and no present layer giving an empty `Doc`.

`doc.go` describes the stack, the file locations, strictness per base layer, the local rule, single post-merge expansion, and that local override files are never disposable for any cleanup.
`pattern/PATTERN-config-strictness.md` states that strictness applies to the base layer, the fill runs down to the template, and a local file refuses unknown keys; `pattern/PATTERN-durable-vs-ephemeral-state.md` states that the local override sits in the mirrored directory, not at the mirrored file path, which is the editor staging copy's.

**ImpactSummary:** re-signs both loaders: twelve module loaders, `loggerconfig.Load`, configcli's hub validation and two test helpers move to a behaviour-preserving transitional geometry.
