# Card 10 — darn and landing layered load

**Create:**
- `plan:internal/darnengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/landingshed#ConfigSpec` -> `func ConfigSpec() configengine.Spec`

**Edit:**
- `internal/darnengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `internal/darnengine#VerifyCommand` -> `func VerifyCommand(dirs configengine.Dirs) (string, error)`
- `internal/landingshed#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `internal/darnengine#TestLoadConfig`
- `internal/darnengine#TestVerifyCommand`
- `internal/landingshed#TestLoadConfig_InvalidKeys`
- `internal/landingshed#TestLoadConfig_NotInitialized`
- `internal/landingshed#TestLoadConfig_PublishVerify`
- `internal/landingshed#TestLoadConfig_TemplateDefaults`
- `internal/loomcli#darnVerifySource`
- `internal/loomcli#loomCLI.darnReviewBudget`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli/wiring.go`

**Uses:**
- `internal/configengine#Spec`
- `internal/configengine#Dirs`
- `internal/fabricengine#ConfigDirs`

**Intent:**
Move the two hub-wide modules with a non-empty local bound onto the hub-wide stack of D1: template, hub file at `<BoardDir>/_lyx/config/`, then the reading worktree's local override restricted to the module's `CategoryNone` keys.
Each loader takes `configengine.Dirs` and loads strictly through `configengine.Load(dirs, ConfigSpec())`, the hub file staying the base layer and its expansion running against the board dir.

- `darnengine.ConfigSpec` is hub-wide with `Categories` as the overview's table lists: `writer`, `writer_timeout_min` and `verify_attempts` none; `verify` executable; `review_max_bounces` guard, since 0 skips Darn-Review and Describe.
  `VerifyCommand` takes the dirs too.
- `landingshed.ConfigSpec` is hub-wide with `conflict`, `conflict_timeout_min`, `describe` and `describe_timeout_min` none; `publish_verify` executable; `require_pr_to_base`, `squash` and `co_authored_by` cross-pair, since each gates or shapes what lands on the parent branch.

The darn wiring loads darn from the board dir alone today (the record's note on `darnwiring.go`); `darnVerifySource`, `wireDarn` and `darnReviewBudget` now pass `fabricengine.ConfigDirs(location)`, so a pair's local `writer` reaches the darn writer and a pair's local `verify` is refused.
`internal/loomcli/wiring.go` loads landing with `fabricengine.ConfigDirs(location)` instead of the bare board dir.

`TestLoadConfig` (darn) gains rows over a temp board dir and anchor: a local `writer` wins over the hub file's, and a local `verify` fails with `ErrInvalid` naming the local file and the key.
`TestLoadConfig_InvalidKeys` (landing) gains a local `conflict` winning, and a local `publish_verify` refused the same way; each loader's one refusal row proves it passes its `ConfigSpec`, and card 13's `TestModules_LocalLayerRefusals` owns the local-layer keys no loader card pins, darn's `review_max_bounces` and landing's `require_pr_to_base` among them, repeating none of `verify`, `publish_verify`, `writer` and `conflict`; the other listed tests take the new argument.

**ImpactSummary:** re-signs the darn and landing loaders and `VerifyCommand`: loomcli's darn and landing wiring pass the pair's dirs; the hub file stays the base layer.
