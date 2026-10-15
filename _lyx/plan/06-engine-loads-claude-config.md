# Card 6 — engine-loads-claude-config

**Create:**
- `plan:internal/shuttleengine/claudeengine#engineWith` -> `func engineWith(t testing.TB, claude Config) *Claude`
- `plan:internal/shuttleengine/claudeengine#loadConfigOrRefuse` -> `func loadConfigOrRefuse(dirs configengine.Dirs) (Config, error)`
- `plan:internal/shuttleengine/claudeengine#engineWithInvalidConfig` -> `func engineWithInvalidConfig(t testing.TB) *Claude`
- `plan:internal/loomcli#claudeConfigWithBinary` -> `func claudeConfigWithBinary(binary string) string`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude`
- `internal/shuttleengine/claudeengine#NewFromConfig` -> `func NewFromConfig(cfg shuttleengine.Config, dirs configengine.Dirs) *Claude`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#Claude.OpenLaunch`
- `internal/shuttleengine/claudeengine#Claude.PromptCacheLifetime`
- `internal/shuttleengine/claudeengine#claudeBinary` -> `func claudeBinary(claude Config) string`
- `internal/shuttleengine/claudeengine#validatePromptCacheTTL`
- `internal/shuttleengine/claudeengine#resolvePromptCacheTTL` -> `func resolvePromptCacheTTL(role string, claude Config) (string, error)`
- `internal/shuttleengine/claudeengine#denyInputs`
- `internal/shuttleengine/claudeengine#standingDenies`
- `internal/shuttleengine/claudeengine#buildDenyNotice` -> `func buildDenyNotice(interactive bool, cfg shuttleengine.Config, claude Config, forkSubagents, allowAgentTool bool) string`
- `internal/shuttleengine/claudeengine#buildSettings` -> `func buildSettings(eventsPathPosix string, interactive bool, cfg shuttleengine.Config, claude Config, forkSubagents, allowAgentTool bool, contextAfterCompaction string) ([]byte, error)`
- `internal/shuttleengine/claudeengine#ttlConfig` -> `func ttlConfig(t testing.TB, defaultTTL string, roles map[string]string) Config`
- `internal/shuttleengine/claudeengine#templateConfig` -> `func templateConfig(t testing.TB) Config`
- `internal/shuttleengine/claudeengine#TestClaudeBinary`
- `internal/shuttleengine/claudeengine#TestOpenLaunch`
- `internal/shuttleengine/claudeengine#TestPrepare_RejectedBeforeArtifacts`
- `internal/shuttleengine/claudeengine#TestPromptCacheLifetime`
- `internal/shuttleengine/claudeengine#TestPrepare_ThreadsSpecIntoLaunchCmds`
- `internal/shuttleengine/claudeengine#TestPrepare_AppendSystemPromptMatchesDenyNotice`
- `internal/shuttleengine/claudeengine#TestPrepare_ResumeSessionID`
- `internal/shuttleengine/claudeengine#TestPrepare_SkillsDeferPromptPointer`
- `internal/shuttleengine/claudeengine#TestPrepare_LogsResolvedPromptCacheTTL`
- `internal/shuttleengine/claudeengine#TestPrepare_PromptLaunchLimit`
- `internal/shuttleengine/claudeengine#TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`
- `internal/shuttleengine/claudeengine#TestBuildSettings_PromptSuggestionOff`
- `internal/shuttleengine/claudeengine#TestBuildSettings_RecordingHooks`
- `internal/shuttleengine/claudeengine#TestBuildSettings_SessionStartHook`
- `internal/shuttleengine/claudeengine#TestBuildSettings_DenyToggleMatrix`
- `internal/shuttleengine/claudeengine#TestBuildSettings_AgentAndBashHooks`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_MatchesInstalledDenies`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_SentenceContent`
- `plan:internal/shuttleengine/claudeengine#TestBuildSettings_MatchesGolden`
- `internal/shuttleengine/claudeengine#recordingHookCommands`
- `internal/shuttleengine/claudeengine#TestEnvFile_DefaultStdinIsDevNull`
- `internal/shuttleengine/claudeengine#TestComposeSend`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengines#constructor`
- `plan:internal/shuttleengines#entry`
- `internal/shuttleengines#registry`
- `internal/shuttleengines#New` -> `func New(cfg shuttleengine.Config, dirs configengine.Dirs) (shuttleengine.EngineSet, error)`
- `internal/shuttleengines#build` -> `func build(cfg shuttleengine.Config, dirs configengine.Dirs, constructors map[string]constructor) (shuttleengine.EngineSet, error)`
- `internal/shuttleengines#TestNew`
- `internal/shuttleengines/doc.go`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#readAgentStates`
- `internal/loomcli#loomCLI.stopStepRuns`
- `internal/loomcli#agentActivityFor`
- `internal/orchcli#Command`
- `internal/shuttlecli#Command`
- `internal/battencli#battenCLI.loadedTacklerDeps`
- `internal/battencli#readAgentActivity`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#readAgentStates`
- `internal/tendercli#newShuttlePair`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#realForkSettingsPath`
- `internal/loomcli#TestNewParentReviewConfig_KeepAliveInterval`
- `internal/loomcli#newWiredPairFixture`
- `internal/loomcli#newBadReedUpFixture`
- `internal/loomcli#TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand`
- `internal/loomcli#TestSmokeDriverStrand_ReentrantAcrossThreeBootstraps`
- `internal/loomcli#TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver`
- `internal/treadleengine_test#TestSmokeJudgeCirclingToyFixture`
- `internal/orchcli#newLiveFixture`
- `internal/burlerengine_test#TestSmokeBurlerRoundToyFixture`
- `internal/burlerengine_test#newClusterSmokeEngine`
- `internal/shuttlecli#newSmokeRunner`
- `internal/seatengine_test#TestSmokeSeatsChairAsksAdvisor`
- `internal/loomengine/plan.go`

**Uses:**
- `plan:internal/shuttleengine/claudeengine#Config`
- `plan:internal/shuttleengine/claudeengine#LoadConfig`
- `plan:internal/shuttleengine/claudeengine#ConfigTemplate`
- `internal/shuttleengines#loadWithLocalEngine`
- `internal/testkit/shuttlefake#Reed`

**Intent:**
Switch the Claude engine onto `claude.yaml`: the engine is told the `configengine.Dirs` of the location it serves and loads its own config in the methods that read it (D2), and every caller of `shuttleengines.New` hands it those `Dirs`.

`Claude` gains the field `dirs configengine.Dirs`, and `NewFromConfig` takes it beside the shuttle config.
`New()` keeps its signature and zero `Dirs`; it stays the engine for reading uses (parsing, auditing, session naming), and its config-reading methods fail with `configengine`'s error since no `Dirs` were told.
`Prepare`, `OpenLaunch` and `PromptCacheLifetime` each call `loadConfigOrRefuse(c.dirs)` first, before any validation or artifact.
`loadConfigOrRefuse` returns `LoadConfig`'s value, and on an error wraps it as `claudeengine: %w; way forward: correct the key with lyx config claude --set or lyx config claude`, keeping `configengine`'s text naming the file and key and its `ErrInvalid` mark, so an invalid `claude.yaml` refuses a run start, a resume (a run start with `ResumeSessionID`) and an open with the file, the key and the way forward.
The reachable input is a `claude.yaml` layer `LoadConfig` refuses, such as a `claude.local.yaml` holding `binary: /x`; `engineWithInvalidConfig` builds an engine over a temp anchor holding exactly that.
No other method and neither constructor loads config, so engine construction, stop, the step-stop sweep, strand removal, the activity and state reads and lock checks never meet a bad `claude.yaml`.
Bound: the load adds one layered read per run start, open and resume, never one per poll, since the polling paths build engines through `New` and never call these three methods.

The loaded `claude` value replaces the shuttle config's Claude fields everywhere the engine reads them: `claudeBinary` reads `Binary`; `resolvePromptCacheTTL` reads `PromptCacheTTL` and `PromptCacheTTLRoles`; `denyInputs` gains a `claude Config` field and the Agent and AskUserQuestion rows of `standingDenies` read its `DenyAgentTool` and `DenyAskUserQuestion`; `buildSettings` and `buildDenyNotice` take the `claude` value beside the shuttle config, which they keep for the python switch until card 7 renames it.
`validatePromptCacheTTL`'s refusal names the new keys, `prompt_cache_ttl` and `prompt_cache_ttl_roles[<role>]`, and its way forward becomes `fix it in claude.yaml with lyx config claude`; the rows asserting the old key names and the `shuttle.yaml` advice in `TestPrepare_RejectedBeforeArtifacts` and `TestPromptCacheLifetime` follow.
The shuttle config's Claude fields stay declared but unread until card 7 removes them.

`shuttleengines`: `constructor` becomes `func(cfg shuttleengine.Config, dirs configengine.Dirs) shuttleengine.Engine` and the `claude` entry calls `claudeengine.NewFromConfig(cfg, dirs)`; `New` and `build` take the `Dirs` and hand them to every engine.
Each caller forms the `Dirs` as it does for its own `shuttle.yaml` load: `fabricengine.ConfigDirs(<location>)` in a hub, and the standalone `configengine.Dirs{Anchor: res.StateDir}` in `burlerCLI.wireStandalone` and `websterCLI.wireStandalone`.
The six `llm`-tier smoke fixtures that build a real `Runner` over `shuttlefake.EngineSet(claudeengine.New())` (in `treadleengine`, `orchcli`, `burlerengine`, `shuttlecli` and `seatengine`) start real runs, which a zero-`Dirs` engine now refuses, so each builds its engine with `claudeengine.NewFromConfig(shuttleCfg, fabricengine.ConfigDirs(<the hub location it loads shuttleCfg from>))`.

The file header of `claudeengine.go` and the doc of `Claude` say the type carries only its settles and clock and every method is a pure function of them; both are rewritten to say the type also carries the told `Dirs`, and that `Prepare`, `OpenLaunch` and `PromptCacheLifetime` read `claude.yaml` through them while every other method stays a pure function of its arguments, settles and clock.

Tests:
- `engineWith` writes `claude` as YAML into a temp anchor's worktree `claude.yaml` and returns `NewFromConfig` over the template shuttle config and that anchor's `Dirs`; `ttlConfig` and `templateConfig` now build a `claudeengine` `Config` from the shipped `claude.yaml` template, so fixtures follow the template.
  Every listed test that called `New().Prepare`, `New().OpenLaunch` or `New().PromptCacheLifetime` with Claude fields on the shuttle config calls them on `engineWith(t, <claude config>)`, and every direct `buildSettings`, `buildDenyNotice`, `claudeBinary` or `resolvePromptCacheTTL` call passes the `claude` value.
- `TestBuildSettings_MatchesGolden` drives the Agent and AskUserQuestion denies through the `claude` value and the python deny through the shuttle config, and still matches the golden file byte for byte.
- `TestPrepare_RejectedBeforeArtifacts`, `TestOpenLaunch` and `TestPromptCacheLifetime` each gain a row over `engineWithInvalidConfig` asserting the call refuses naming `claude.local.yaml`, `binary` and `lyx config claude`, and, for `Prepare`, that no artifact was written.
- `TestNew` passes the `Dirs` and its constructor fakes take them, and gains the row `InvalidEngineConfig`: a temp anchor whose `claude.local.yaml` holds `binary: /x`, beside a `shuttle.local.yaml` holding `claude_prompt_cache_ttl_roles` with two roles.
  The row loads the shuttle config through `shuttleengine.LoadConfig` over that anchor, asserts `New` builds the set and looks up the default engine, and builds a `shuttleengine.NewRunner` over a `shuttlefake.Reed` holding one live strand and that set.
  It asserts `StopStrand` on the strand succeeds and the fake recorded its removal; that `ReadAgentActivity`, `ReadAgentWaits` and `ReadSessionStates` over the anchor return no error; and that `Runner.Start` of a spec refuses naming `claude.local.yaml` and `binary`.
  So a bad `claude.yaml` refuses a run start end to end while stop, strand removal and the activity and state reads still work.
- `TestComposeSend` builds its engines with `NewFromConfig` and zero `Dirs`, since it reads no config.
- `TestNewParentReviewConfig_KeepAliveInterval` writes each row's TTL keys into a temp anchor's `claude.yaml` and builds the engine with `NewFromConfig` over it.
- `realForkSettingsPath` drops its shuttle-config deny overrides, which the shipped `claude.yaml` template already sets, and prepares through `NewFromConfig` over a temp anchor's `Dirs`, which degrade to that template.
- The `loomcli` smoke and integration fixtures that pointed shuttle's `claude:` key at a stub or a nonexistent path seed `claude.yaml` too, as `claudeConfigWithBinary(<path>)`: the shipped `claude.yaml` template with its `binary:` line pointed at the path, failing loudly on template drift; otherwise those fixtures would launch the real `claude` on `PATH`.

Docs: `claudeengine`'s `doc.go` says Prepare, OpenLaunch and PromptCacheLifetime read `claude.yaml` and names its keys where it named `shuttle.yaml`'s; `shuttleengines`'s `doc.go` says `New` hands every engine the `Dirs` it serves; `internal/loomengine/plan.go`'s header comment names `claude.yaml`'s `deny_agent_tool` in place of the shuttle key.

**ImpactSummary:** Re-signs `shuttleengines.New` and `claudeengine.NewFromConfig`, covering their callers across burlercli, loomcli, orchcli, shuttlecli, battencli, tendercli and webstercli; Claude's binary, Agent and AskUserQuestion denies and TTLs now come from `claude.yaml`, the python switch still from `shuttle.yaml`.
