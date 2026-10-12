# Card 5 — shuttle layered load

**Create:**
- `internal/shuttleengine#ConfigSpec`

**Edit:**
- `internal/shuttleengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `internal/shuttleengine_test#TestLoadConfig_BackgroundShellWaitMin`
- `internal/shuttleengine_test#TestLoadConfig_EnvOverride`
- `internal/shuttleengine_test#TestLoadConfig_ModuleArgIsThreadedThrough`
- `internal/shuttleengine_test#TestLoadConfig_PromptCacheTTLKeys`
- `internal/shuttleengine_test#TestLoadConfig_SubmitSettleKeys`
- `internal/shuttleengine_test#TestLoadConfig_TemplateDefaultsResolve`
- `internal/shuttleengine_test#TestLoadConfig_UninitializedFallsBackToTemplate`
- `internal/shuttleengine/claudeengine#ttlConfig`
- `internal/battencli/wire.go`
- `internal/burlercli/wiring.go`
- `internal/burlerengine/smoke_cluster_test.go`
- `internal/burlerengine/smoke_round_test.go`
- `internal/loomcli/reviewstatus.go`
- `internal/loomcli/smoke_attachprobe_test.go`
- `internal/loomcli/smoke_gate_test.go`
- `internal/loomcli/smoke_parentreview_test.go`
- `internal/loomcli/wiring.go`
- `internal/orchcli/cli.go`
- `internal/orchcli/smoke_cycle_test.go`
- `internal/orchcli/smoke_live_test.go`
- `internal/seatengine/smoke_seats_test.go`
- `internal/shuttlecli/cli.go`
- `internal/shuttlecli/smoke_interrupt_test.go`
- `internal/treadleengine/smoke_judge_test.go`
- `internal/webstercli/smoke_test.go`
- `internal/webstercli/wiring.go`

**Uses:**
- `internal/configengine#Spec`
- `internal/configengine#Dirs`
- `internal/fabricengine#ConfigDirs`

**Intent:**
Move shuttle onto the hub stack the way card 4 moved reed: `LoadConfig` takes `configengine.Dirs` and loads through `configengine.LoadOrTemplate(dirs, ConfigSpec())`.
`ConfigSpec` returns shuttle's `Spec`: name `shuttle`, its template, `ConfigOpenMaps()`, not hub-wide, and `Categories` exactly as the overview's "Per-key classification" table lists shuttle's keys, so `claude` is executable, `run_dir` cross-pair, each `claude_deny_*` key a guard, and every other key, the open map `claude_prompt_cache_ttl_roles` included, none.

Callers pass the geometry they hold, by the rule card 4 states: `fabricengine.ConfigDirs(location)` with a location, `configengine.Dirs{Anchor: stateDir}` standalone, `configengine.Dirs{Anchor: dir}` in a test.

`TestLoadConfig_TemplateDefaultsResolve` gains two rows over a temp board dir and anchor that pin the wiring: a `run_timeout_min` set only in the repo default resolves beneath a worktree file lacking it, and a local `claude_deny_python` fails the load with `ErrInvalid` naming the local file and the key, proving the loader passes its `ConfigSpec`; card 13's `TestModules_LocalLayerRefusals` owns the local-layer keys no loader card pins, shuttle's open-map entry among them, and never repeats `claude_deny_python`.

**ImpactSummary:** re-signs `shuttleengine.LoadConfig`: every caller across the CLIs, battencli and the tmux smoke tests passes told dirs; no behaviour changes outside a hub.
