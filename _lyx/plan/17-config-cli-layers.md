# Card 17 — config CLI layers

**Create:**
- `plan:internal/configcli#configDirs.layerFile` -> `func (d configDirs) layerFile(module configreg.Module) (string, configengine.Layer, error)`
- `plan:internal/configcli#TestDispatchLayers` -> `func TestDispatchLayers(t *testing.T)`

**Edit:**
- `internal/configengine#Set` -> `func Set(dirs Dirs, spec Spec, layer Layer, pairs []yamlengine.KV) ([]string, error)`
- `internal/configengine#Edit` -> `func Edit(dirs Dirs, spec Spec, layer Layer, edit EditorFunc) error`
- `internal/configengine_test#TestSet`
- `internal/configengine_test#TestEdit`
- `internal/configcli#configDirs`
- `internal/configcli#dispatch` -> `func dispatch(l *lyxcwd.Location, out io.Writer, args []string, edit configengine.EditorFunc, sync syncFunc, commit hubCommitFunc, printOnly bool, setFlags []string, layer string, hubWriteGuard func() (string, error)) int`
- `internal/configcli#runConfig` -> `func runConfig(ctx context.Context, out io.Writer, args []string, printOnly bool, setFlags []string, layer string) int`
- `internal/configcli#Command`
- `internal/configcli#buildConfigLong`
- `internal/configcli#printModule`
- `internal/configcli#printAll`
- `internal/configcli#editOne`
- `internal/configcli#setModule`
- `internal/configcli#setHubWide`
- `internal/configcli#validateHubFile`
- `internal/configcli#menu`
- `internal/configcli#TestEditOne`
- `internal/configcli#TestDispatchSet`
- `internal/configcli#TestDispatchHubWideBoard`
- `internal/configcli#TestPrint`
- `internal/configcli#TestMenu`
- `internal/configcli#TestConfigLong`
- `internal/configcli#runMenuWith`
- `internal/configcli#TestConfigOverRealHub`
- `internal/configcli#TestConfigReconcileInGitRepo`
- `internal/configcli/doc.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Delete:**
- `internal/configcli#configDirs.baseFor`

**Uses:**
- `plan:internal/configengine#LayerFile`
- `plan:internal/configengine#BaseLayer`
- `plan:internal/configengine#ValidateLocal`
- `plan:internal/yamlengine#SetSparse`
- `internal/configreg#Module`

**Intent:**
Give `lyx config <module>` D7's layer selection.
`Command` gains `--layer default|worktree|local`, threaded through `runConfig` and `dispatch` into a new `layer` field of `configDirs`, which `dispatch` validates against the three names (empty meaning the module's own default: the worktree file of a per-worktree module, the hub file of a hub-wide one, as today).
`configDirs.layerFile` replaces `baseFor`: it returns the selected layer's file through `configengine.LayerFile(dirs, module.Spec(), layer)` and refuses `--layer worktree` for a hub-wide module with a message naming `--layer default` from the prime and `--layer local` as the ways forward.

- `--layer default` writes `<BoardDir>/_lyx/config/<module>.yaml`, the repo default of a per-worktree module and the hub file of a hub-wide one, through the hub-wide write path unchanged: the guard (refused from a non-prime worktree and from a slug-named session), the board write lock, strict validation by `validateHubFile`, which loads the board layer alone through `configengine.Load(configengine.Dirs{Board: boardDir}, module.Spec())`, restore on failure, and the commit on Bolt and push.
- `--layer worktree` (and no flag, for a per-worktree module) writes the worktree file and syncs fabric as today.
- `--layer local` writes `configengine.LocalFile` of the anchor, never commits and runs no fabric sync, and refuses a key D5 would refuse at load, leaving the file as it was.

`configengine.Set` and `Edit` take the dirs, the spec and the layer: the base layer keeps today's scaffold from the template and `SetValues`; any layer above it is sparse, so `Set` writes only the set keys through `yamlengine.SetSparse` and `Edit` scaffolds an absent file empty, never from the template (D2); a local write is checked with `ValidateLocal` and a refused one restores the file's previous bytes, or removes a file the call created.
So `lyx config loom --set review_max_bounces=2` in a hub pair with no worktree file writes a file holding only that key.
`--print` prints the selected layer's file verbatim; `printAll` keeps printing each module's own default file.
`menu` marks each module with the layers that hold a file, as `loom (default, local)`, or `(template)` when none does, replacing the `(configured)` and `(default)` marks.
`buildConfigLong` and `doc.go` document `--layer`, the sparse writes, the local bound and the menu marks.
`contracts/specs/refusal-spec.md`'s "hub-wide config write refused" row widens its trigger to a `--layer default` write of any module.

`TestDispatchLayers` is table-driven over a hub fixture: `--layer bogus` is refused naming the three layers and writes nothing; `--layer default --set` on loom writes and commits the repo default from the prime, and is refused from a pair and from a slug-named session; `--layer worktree` on darn is refused; `--layer local --set writer=...` on darn writes the local file uncommitted, while `--layer local --set verify=...` is refused naming the key and leaves no file; a `--set` into an absent pair worktree file writes only that key; `--print --layer local` prints the local file.
`TestSet` and `TestEdit` gain the sparse rows (only the set key written, an empty scaffold) and a refused local write restored; `TestMenu` asserts the new marks; the other listed tests take the new arguments.
The budget row for `internal/configcli` rises by one.

**ImpactSummary:** re-signs `configengine.Set`, `Edit` and configcli's dispatch: every write path of `lyx config` becomes layer-aware, the menu's marks change, and one refusal-spec row widens.
