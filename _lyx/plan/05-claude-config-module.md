# Card 5 — claude-config-module

**Create:**
- `internal/shuttleengine/claudeengine/template.yaml`
- `internal/shuttleengine/claudeengine#configTemplate`
- `internal/shuttleengine/claudeengine#Config`
- `internal/shuttleengine/claudeengine#ConfigTemplate`
- `internal/shuttleengine/claudeengine#ConfigOpenMaps`
- `internal/shuttleengine/claudeengine#ConfigSpec`
- `internal/shuttleengine/claudeengine#LoadConfig`
- `internal/shuttleengine/claudeengine#TestLoadConfig`
- `internal/shuttleengines#ConfigModule`
- `internal/shuttleengines#entry`
- `internal/shuttleengines#ConfigModules`

**Edit:**
- `internal/shuttleengines#registry`
- `internal/shuttleengines#New`
- `internal/shuttleengines#Names`
- `internal/shuttleengines#allowedImports`
- `internal/shuttleengines/doc.go`
- `internal/configreg#Modules`
- `internal/configreg#registeredPackages`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `cmd/lyx#configStrictnessDegradingSet`
- `internal/shuttleengine/claudeengine/doc.go`
- `pattern/PATTERN-config-strictness.md`
- `pattern/PATTERN-shuttle-provider-seam.md`

**Intent:**
Declare `claude.yaml`, a layered config module owned and loaded by `claudeengine` (D1), and register it in `configreg` through `internal/shuttleengines` so `configreg` stays engine-blind (D2).
No code reads the module yet; card 6 switches the engine onto it.

`template.yaml`, embedded as `configTemplate` and returned by `ConfigTemplate`, carries today's values and comments of the five Claude keys with the `claude_` prefix dropped:
`binary: ${env:LYX_SHUTTLE_CLAUDE:-}` with today's `claude:` comment, `deny_agent_tool: true`, `deny_ask_user_question: true`, `prompt_cache_ttl: 5m` and `prompt_cache_ttl_roles` holding `driver: 1h` and `webster: 1h`, the comments naming `prompt_cache_ttl_roles` and `claude.yaml` where today's name the old keys.
`Config` mirrors them as `Binary`, `DenyAgentTool`, `DenyAskUserQuestion`, `PromptCacheTTL` and `PromptCacheTTLRoles map[string]string`.
`ConfigOpenMaps` returns `prompt_cache_ttl_roles`.
`ConfigSpec` returns the spec named `claude`, per-worktree, with categories carried over from shuttle's classification: `binary` Executable, `deny_agent_tool` and `deny_ask_user_question` Guard, `prompt_cache_ttl` and `prompt_cache_ttl_roles` None, so the local layer admits exactly what it admitted before.
`LoadConfig` loads through `configengine.LoadOrTemplate`, the degrading policy `shuttle.yaml` uses, stacking the template, the repo default `<Board>/_lyx/config/claude.yaml`, the worktree file and `.lyx/config/claude.local.yaml`, and unmarshals into `Config`; it is told its `Dirs` and derives none (`PATTERN-told-geometry`).
TTL values stay validated where a run starts, as today.

In `shuttleengines`, `ConfigModule` carries one engine's config module as plain data: `Name`, `Template func() string`, `OpenMaps []string`, `Migrate func(existing []byte) ([]byte, []string, error)` (nil for an engine without a migrate hook, as Claude's is), `Spec func() configengine.Spec` and `CheckLoad func(configengine.Dirs) error`.
`entry` pairs a registered engine's `constructor` with its `ConfigModule`, and `registry` becomes `map[string]entry`, the `claude` entry carrying `claudeengine.ConfigTemplate`, `ConfigOpenMaps()`, `ConfigSpec` and a `CheckLoad` over `claudeengine.LoadConfig`.
`New` and `Names` read the constructors and names out of the entries; `build` and its refusals are unchanged.
`ConfigModules` returns every registered engine's module sorted by name, so a later engine adds its module with its registry entry alone.
`allowedImports` admits `internal/configengine`.

`configreg.Modules` appends a `Module` per `shuttleengines.ConfigModules()` entry (name, template, open maps, migrate hook, `CheckLoad` and `Spec`), then keeps the whole list in name order, so `claude` lists between `burler` and `darn` and `configreg` imports `shuttleengines`, never `claudeengine`.
`registeredPackages` also walks `internal/shuttleengines/shuttleengines.go` and records the package of every `ConfigTemplate` selector it holds, resolving import names as it does for `configreg.go`, so `TestRegistration_MatchesDeclarers` counts `claudeengine` as registered through the registry.
`TestModules_EveryTemplateKeyClassified` covers `claude` and pins its `LocalDenied` as `binary`, `deny_agent_tool` and `deny_ask_user_question`.
`claudeengine` now calls `configengine.LoadOrTemplate`, so `configStrictnessDegradingSet` gains `internal/shuttleengine/claudeengine` and `PATTERN-config-strictness`'s background lists `claudeengine` among the degrading callers.

`TestLoadConfig` pins the acceptance rows over temp dirs: the template on an absent base; a repo default and a worktree file stacked, the worktree key winning; a local file holding `prompt_cache_ttl` and two `prompt_cache_ttl_roles` entries accepted; a local `binary`, `deny_agent_tool` and `deny_ask_user_question` each refused naming `claude.local.yaml` and the key; `LYX_SHUTTLE_CLAUDE` set resolving `binary` to its value, the one row that sets the environment, so the test does not run in parallel.
The `claudeengine` budget row raised in card 1 covers the new test, since card 2 removed five.

Docs: `shuttleengines`'s `doc.go` says each registry entry carries its engine's config module and `ConfigModules` lists them for `configreg`; `claudeengine`'s `doc.go` describes `claude.yaml`; `PATTERN-shuttle-provider-seam`'s background says an engine's config module reaches `configreg` only through `shuttleengines`.

**ImpactSummary:** Adds the `claude` module to every config surface (`lyx config`, reconcile, the registry fingerprint) through `shuttleengines`; `New`'s behavior and every existing module are unchanged, and nothing reads `claude.yaml` yet.
