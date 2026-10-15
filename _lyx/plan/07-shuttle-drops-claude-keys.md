# Card 7 — shuttle-drops-claude-keys

**Create:**
- `plan:internal/shuttleengine#renameDenyPython` -> `func renameDenyPython(existing []byte) ([]byte, string, error)`
- `plan:internal/shuttleengine_test#TestLoadConfig_RetiredClaudeKeys` -> `func TestLoadConfig_RetiredClaudeKeys(t *testing.T)`
- `plan:internal/shuttleengine_test#TestMigrateConfig_RenamesDenyPython` -> `func TestMigrateConfig_RenamesDenyPython(t *testing.T)`

**Delete:**
- `internal/shuttleengine#ConfigOpenMaps`
- `internal/shuttleengine_test#TestLoadConfig_PromptCacheTTLKeys`
- `internal/shuttleengine_test#TestLoadConfig_EnvOverride`

**Edit:**
- `internal/shuttleengine#Config`
- `internal/shuttleengine#ConfigSpec`
- `internal/shuttleengine#MigrateConfig`
- `internal/shuttleengine#ConfigTemplate`
- `internal/shuttleengine#Spec`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine_test#TestLoadConfig_TemplateDefaultsResolve`
- `internal/shuttleengine_test#TestLoadConfig_UninitializedFallsBackToTemplate`
- `internal/configreg#Modules`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestLookup`
- `internal/configreg#TestPrepareRunSettings`
- `internal/shuttleengine/claudeengine#shellGuardInstalled`
- `internal/shuttleengine/claudeengine#TestBuildSettings_DenyToggleMatrix`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_MatchesInstalledDenies`
- `internal/shuttleengine/claudeengine#TestPrepare_AppendSystemPromptMatchesDenyNotice`
- `internal/shuttleengine/claudeengine#TestBuildSettings_MatchesGolden`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/testkit/envkit#nilLegal`
- `internal/loomcli#providerlessShuttleConfig`
- `internal/loomcli#driverShuttleConfig`
- `internal/loomcli#integrationDriverShuttleConfig`
- `docs/overview.md`

**Uses:**
- `plan:internal/configengine#retiredCovering`
- `internal/shuttleengine#LoadConfig`

**Intent:**
`shuttle.yaml` loses every Claude key (D3): the python switch stays as the neutral `deny_python`, and the five keys `claude.yaml` now owns become retired keys, so an unreconciled layer still loads (D4).

`Config` drops `Claude`, `ClaudeDenyAgentTool`, `ClaudeDenyAskUserQuestion`, `ClaudePromptCacheTTL` and `ClaudePromptCacheTTLRoles`, and `ClaudeDenyPython` becomes `DenyPython` with the YAML key `deny_python`.
`template.yaml` drops the five keys' lines and writes `deny_python: true` with a comment that no longer names Claude: it switches the python guard of `internal/shellguard`'s table for every engine, on by default, off in a Python target repo.
`ConfigSpec` classifies `deny_python` Guard, drops the five keys from `Categories` and drops its open maps, and sets `RetiredKeys` to `claude` Executable, `claude_deny_agent_tool` Guard, `claude_deny_ask_user_question` Guard, `claude_prompt_cache_ttl` None and `claude_prompt_cache_ttl_roles` None, the categories each held.
`ConfigOpenMaps` is deleted with its one remaining caller, `configreg.Modules`' shuttle entry, which carries no open maps.
`claude_deny_python` is not retired: `MigrateConfig` renames it.

`MigrateConfig` runs `renameDenyPython` after its poll-floor step: in a present document holding `claude_deny_python` and no `deny_python`, it renames the key in place, keeping its value, its position and its line comment, through yaml node editing, and returns the line `claude_deny_python: renamed to deny_python`; a document holding neither key comes back unchanged with no line; a document already holding `deny_python` beside `claude_deny_python` keeps `deny_python`, drops `claude_deny_python` and returns the line `claude_deny_python: <value> dropped, deny_python already set`, so the dropped value is reported rather than pruned silently.
It runs wherever `MigrateConfig` runs today: every reconcile layer and a task's run settings, where `PrepareRunSettings` then refuses the renamed `deny_python` as a Guard key, a `RunSettingsError` for module `shuttle` whose cause names `deny_python`.

`shellGuardInstalled` reads `cfg.DenyPython`.
The claudeengine tests that set `ClaudeDenyPython` set `DenyPython`, and the golden test still matches byte for byte.
`envkit`'s `nilLegal` drops `WebsterDeps.ShuttleCfg.ClaudePromptCacheTTLRoles`, a field that no longer exists.
The three `loomcli` shuttle fixtures drop their `claude:` line rewrite, since card 6 moved the binary into the `claude.yaml` fixtures; `driverShuttleConfig` keeps its drift check on `startup_timeout_s` alone.

Tests:
- `TestLoadConfig_TemplateDefaultsResolve` asserts `DenyPython` true and that the template holds no top-level key starting with `claude`; `TestLoadConfig_UninitializedFallsBackToTemplate` asserts `DenyPython` in place of the Agent deny.
- `TestLoadConfig_RetiredClaudeKeys` replaces `TestLoadConfig_PromptCacheTTLKeys`, whose TTL resolution `claudeengine`'s `TestLoadConfig` now covers: a `shuttle.local.yaml` holding `claude_prompt_cache_ttl_roles` with two roles loads, as does one holding `claude_prompt_cache_ttl: {a: 1}`; a local `claude` and a local `claude_deny_agent_tool` are each refused naming the key; a worktree `shuttle.yaml` holding all five retired keys and `claude_deny_python: false` loads with `DenyPython` true, the open-risk behaviour before a walk renames the key.
  `shuttleengines`' `TestNew` row `InvalidEngineConfig` from card 6 loads a `shuttle.local.yaml` holding `claude_prompt_cache_ttl_roles` and then stops and removes a strand; from this card on that key is retired, so the row pins stop and strand removal over an unreconciled local layer, and `TestLoadConfig_RetiredClaudeKeys` asserts the merged document `configengine.LoadOrTemplate` resolves for `ConfigSpec()` over that layer holds no `claude_prompt_cache_ttl_roles` key.
- `TestMigrateConfig_RenamesDenyPython` pins the rename keeping value and comment, the no-op over a document without the key, the document holding both keys keeping `deny_python: true` and reporting the dropped `claude_deny_python: false`, and that a second pass over the output changes nothing and reports no line.
- `TestLoadConfig_EnvOverride` is deleted: its only subject, `LYX_SHUTTLE_CLAUDE` reaching the `claude` key, is `claudeengine`'s `TestLoadConfig` row now.
- `TestModules_EveryTemplateKeyClassified` pins shuttle's `LocalDenied` as `deny_python` and `run_dir`; `TestLookup`'s shuttle row expects no open maps; `TestPrepareRunSettings` gains the row `shuttle: {claude_deny_python: true}` refused as above.
The shuttleengine budget row is unchanged: two tests leave and two arrive.

Docs: `ConfigTemplate`'s comment, the `Spec` field comments that named the `claude_deny_*` keys, `shuttleengine`'s `doc.go` (which said `shuttle.yaml` carries the TTL keys), `claudeengine`'s `doc.go` (the python switch is `shuttle.yaml`'s `deny_python`) and `docs/overview.md`'s shuttle line, which names `claude.yaml`'s `deny_agent_tool`, `prompt_cache_ttl` and `prompt_cache_ttl_roles` and `shuttle.yaml`'s `deny_python` in place of the old keys.

**ImpactSummary:** Removes the Claude fields from `shuttleengine.Config` (read only by `claudeengine` until card 6, and now by nobody) and renames the python switch; unreconciled layers keep loading through the retired keys, and reconcile prunes the old keys until card 8 moves them first.
