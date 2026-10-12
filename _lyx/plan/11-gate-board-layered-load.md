# Card 11 — gate and board layered load

**Create:**
- `plan:internal/gateslot#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/boardengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`

**Edit:**
- `internal/gateslot#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs) (Config, error)`
- `internal/boardengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `internal/boardengine#OpenHub`
- `internal/gateslot#TestLoadConfig`
- `internal/boardengine_test#TestLoadConfig_LabelLists`
- `internal/boardengine_test#TestLoadConfig_LabelMaps`
- `internal/boardengine_test#TestLoadConfig_LabelsRefused`
- `internal/boardengine_test#TestLoadConfig_NotInitialized`
- `internal/boardengine_test#TestLoadConfig_ScalarKeysAndTemplateDefaults`
- `internal/hubgeom#GateSlots`
- `internal/ideengine#Menu`
- `internal/boardcli#Command`
- `internal/battencli#fourRowRunForRecipe`
- `internal/battencli#seedBoardTask`
- `internal/pairteardown_test#holdBoardEntry`
- `internal/planglyph#unusableGatePool`
- `internal/verifytree#TestVerify_SlotGate`
- `internal/websterengine#TestRerunCardVerifies_Slotted`

**Uses:**
- `plan:internal/configengine#Spec`
- `plan:internal/configengine#Dirs`
- `plan:internal/fabricengine#ConfigDirs`

**Intent:**
Move gate and board, two hub-wide modules whose every key is closed to the local layer, onto the hub-wide stack: each loader takes `configengine.Dirs` and loads strictly through `configengine.Load(dirs, ConfigSpec())`.
`gateslot.ConfigSpec` and `boardengine.ConfigSpec` are hub-wide, with `Categories` as the overview's table lists: gate's `slots`, `go_parallel` and `cli_wait_sec` cross-pair, one hub-wide budget every pair shares; board's `readme`, `design_prefix`, `types` and `labels` cross-pair, the one board's vocabulary; board keeps `ConfigOpenMaps()`.
So a pair's `gate.local.yaml` or `board.local.yaml` holding any key fails the load naming the file and the key (D5), wherever a reading worktree is told.

A caller holding a location passes `fabricengine.ConfigDirs(location)`: `hubgeom.GateSlots`, `ideengine.Menu` and the boardcli prologue.
`OpenHub`, which is told only a hub path and serves callers with no reading worktree, keeps its signature and passes `configengine.Dirs{Board: BoardDir(hubPath)}`, with no anchor and so no local layer; its doc says so.
Tests pass `configengine.Dirs{Board: dir}` where they load a hub file and `configengine.Dirs{Anchor: dir}` where they load a standalone one.

`TestLoadConfig` (gate) gains a row over a temp board dir and anchor in which a local `slots` fails with `ErrInvalid` naming the local file and the key, and `TestLoadConfig_LabelsRefused` (board) gains the same for a local `labels` entry; the other listed tests take the new argument.

**ImpactSummary:** re-signs the gate and board loaders: hubgeom, ideengine and boardcli pass told dirs, `OpenHub` reads the hub file with no local layer, and fixtures in five test packages follow.
