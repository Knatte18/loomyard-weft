# Card 4 — reed layered load

**Create:**
- `plan:internal/reedengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`

**Edit:**
- `internal/reedengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `internal/reedengine_test#TestLoadConfig_EnvOverride`
- `internal/reedengine_test#TestLoadConfig_MissingSegmentColorsTakeTemplateDefaults`
- `internal/reedengine_test#TestLoadConfig_ModuleArgIsThreadedThrough`
- `internal/reedengine_test#TestLoadConfig_TemplateDefaultsResolve`
- `internal/reedengine_test#TestLoadConfig_UninitializedFallsBackToTemplate`
- `internal/reedengine_test#TestLoadConfig_UnknownKeysFromAnOlderConfigAreIgnored`
- `internal/reedengine#TestDeadSelvagePaneIsHealedByUpWithoutCorruptingLayout`
- `internal/reedengine#TestDisplayMessageDoesNotErrorForAnAbsentSession`
- `internal/reedengine#TestExactSessionTargetsNeverPrefixMatchSiblings`
- `internal/reedengine#TestMultiplexerContract`
- `internal/reedengine#TestRemoveStrand_SoleStrandEmptiesSessionSucceeds`
- `internal/reedengine#TestSelvageNeverGetsZeroHeightLayoutCell`
- `internal/reedengine#TestSessionNameRewriteIsSilentAndExactTargetsMissIt`
- `internal/reedengine#newColdScratchEngine`
- `internal/reedengine#newIntegrationEngine`
- `internal/reedengine#newReviveHub`
- `internal/reedengine#newHubEngine`
- `internal/battencli/cli.go`
- `internal/battencli/wire.go`
- `internal/burlercli/wiring.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `internal/burlerengine/smoke_round_test.go`
- `internal/hubgeom/spawnorder.go`
- `internal/loomcli/smoke_parentreview_test.go`
- `internal/loomcli/smoke_test.go`
- `internal/loomcli/wiring.go`
- `internal/orchcli/cli.go`
- `internal/orchcli/smoke_cycle_test.go`
- `internal/orchcli/smoke_live_test.go`
- `internal/pairteardown/teardown.go`
- `internal/pairteardown/teardown_integration_test.go`
- `internal/reedcli/cli.go`
- `internal/reedcli/cli_integration_test.go`
- `internal/reedcli/entersession_integration_test.go`
- `internal/reedcli/list.go`
- `internal/reedcli/naming_integration_test.go`
- `internal/reedcli/unlessname_integration_test.go`
- `internal/reedcli/watchdog.go`
- `internal/reedcli/watchdog_integration_test.go`
- `internal/reedcli/watchdogreap_integration_test.go`
- `internal/seatengine/smoke_seats_test.go`
- `internal/shuttlecli/cli.go`
- `internal/shuttlecli/smoke_interrupt_test.go`
- `internal/standalonegeom/naming_integration_test.go`
- `internal/treadleengine/smoke_judge_test.go`
- `internal/webstercli/cli_tmux_test.go`
- `internal/webstercli/wiring.go`

**Uses:**
- `plan:internal/configengine#Spec`
- `plan:internal/configengine#Dirs`
- `plan:internal/fabricengine#ConfigDirs`

**Intent:**
Move reed onto the hub stack: `LoadConfig` takes the told `configengine.Dirs` and loads through `configengine.LoadOrTemplate(dirs, ConfigSpec())`, so a key in the repo default resolves beneath a sparse worktree file and a local override applies within D4's bound.
`ConfigSpec` returns reed's `Spec`: name `reed`, the GOOS template, no open maps, not hub-wide, and `Categories` exactly as the overview's "Per-key classification" table lists reed's keys, so `tmux` and `shell` are executable, `tmux_conf` a sourced file, `debug_log` and `mouse` cross-pair, and every other key none.

Every caller passes the geometry it already holds: a caller with a `*lyxcwd.Location` passes `fabricengine.ConfigDirs(location)`, a standalone caller holding only a state dir passes `configengine.Dirs{Anchor: stateDir}`, and a test passes `configengine.Dirs{Anchor: dir}` unless it pins the hub stack.
The production callers in hubgeom, pairteardown, reedcli (the watchdog included) and the CLI wirings already hold the location whose anchor they pass today.

`TestLoadConfig_TemplateDefaultsResolve` gains two rows over a temp board dir and anchor that pin the wiring: a `width` set only in the repo default resolves beneath a worktree file that lacks it, and a local `shell` fails the load with `ErrInvalid` naming the local file and the key, proving the loader passes its `ConfigSpec`; card 13's `TestModules_LocalLayerRefusals` owns the local-layer keys no loader card pins, reed's `mouse` among them, and never repeats `shell`.

**ImpactSummary:** re-signs `reedengine.LoadConfig`: every caller across the CLIs, hubgeom, pairteardown and the tmux smoke tests passes told dirs; no behaviour changes outside a hub.
