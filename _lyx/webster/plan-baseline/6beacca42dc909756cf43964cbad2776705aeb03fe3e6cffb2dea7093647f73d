# Card 6 — orch, webster and logger layered load

**Create:**
- `plan:internal/orchengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/websterengine#ConfigSpec` -> `func ConfigSpec() configengine.Spec`
- `plan:internal/loggerconfig#ConfigSpec` -> `func ConfigSpec() configengine.Spec`

**Edit:**
- `internal/orchengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `internal/websterengine#LoadConfig` -> `func LoadConfig(dirs configengine.Dirs, module string) (Config, error)`
- `internal/loggerconfig#Load` -> `func Load(dirs configengine.Dirs) (logger.RetentionBounds, error)`
- `internal/orchengine_test#TestLoadConfig_CycleMode`
- `internal/orchengine_test#TestLoadConfig_InvalidFileErrors`
- `internal/orchengine_test#TestLoadConfig_PermissionMode`
- `internal/orchengine_test#TestLoadConfig_PresentFileOverrides`
- `internal/orchengine_test#TestLoadConfig_TemplateResolvesWithNoFile`
- `internal/websterengine_test#TestLoadConfig`
- `internal/loggerconfig_test#TestLoad_InvalidValueErrorsNamingKey`
- `internal/loggerconfig_test#TestLoad_ResolvesBounds`
- `cmd/lyx/exitsweep.go`
- `internal/loomcli/wiring.go`
- `internal/orchcli/cli.go`
- `internal/webstercli/wiring.go`

**Uses:**
- `plan:internal/configengine#Spec`
- `plan:internal/configengine#Dirs`
- `plan:internal/fabricengine#ConfigDirs`

**Intent:**
Move the three remaining degrading per-worktree modules onto the hub stack, each as card 4 moved reed: the loader takes `configengine.Dirs` and loads through `configengine.LoadOrTemplate(dirs, ConfigSpec())`, and `ConfigSpec` returns the module's `Spec` with `Categories` exactly as the overview's "Per-key classification" table lists its keys, every one of them none for orch, webster and logger.

`loggerconfig.Load` loses its anchor-only argument, so an early load reads the repo default too (the record's note on degrading callers that load early).
`exitSweepBounds` in `cmd/lyx/exitsweep.go` holds only the sink's anchor: it resolves that anchor's location with `lyxcwd.ResolveWorktree` and passes `fabricengine.ConfigDirs`, falling back to `configengine.Dirs{Anchor: anchor}` when resolution fails, and keeps naming `loggerconfig.ConfigPath(anchor)` in its Warn.
The CLI callers pass `fabricengine.ConfigDirs(location)` or, standalone, `configengine.Dirs{Anchor: stateDir}`.

The knob-validation comment in `websterengine.LoadConfig` still says a webster.yaml carrying only the role keys leaves the knobs at Go's zero value; it is corrected to say the template fill supplies a missing knob and the check refuses an explicit non-positive value.

Each module's existing load test gains one hub-stack row over a temp board dir and anchor, a key set only in the repo default resolving beneath a worktree file lacking it, in `TestLoadConfig_PresentFileOverrides`, `TestLoadConfig` and `TestLoad_ResolvesBounds`; the other listed tests only take the new argument.

**ImpactSummary:** re-signs three loaders: orchcli, webstercli, loomcli and the exit sweep pass told dirs; no behaviour changes outside a hub.
