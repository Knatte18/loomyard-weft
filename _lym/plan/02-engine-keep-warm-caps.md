# Card 2 — engine-keep-warm-caps

**Edit:**
- `internal/shuttleengine#Engine`
- `internal/shuttleengine/claudeengine#Config`
- `internal/shuttleengine/claudeengine#ConfigSpec`
- `internal/shuttleengine/claudeengine#ConfigOpenMaps`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `//internal/shuttleengine/claudeengine/template.yaml`
- `//internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengines#registry`
- `internal/shuttleengine/claudeengine#TestLoadConfig`
- `internal/shuttleengine/claudeengine#TestPrepare_RejectedBeforeArtifacts`
- `internal/shuttleengine/claudeengine#ttlConfig`

**Extends:**
- `//internal/shuttleengines/doc.go`

**Create:**
- `plan:internal/shuttleengine/claudeengine#Claude.KeepWarmCaps` -> `func (c *Claude) KeepWarmCaps(role string) (time.Duration, time.Duration, error)`
- `plan:internal/shuttleengine/claudeengine#ValidateKeepWarm` -> `func ValidateKeepWarm(claude Config) error`
- `plan:internal/shuttleengine/claudeengine#longKeepWarmCap` -> `const longKeepWarmCap = bashMaxTimeout - 5*time.Minute`
- `plan:internal/testkit/shuttlefake#Engine.KeepWarmCaps` -> `func (e *Engine) KeepWarmCaps(role string) (time.Duration, time.Duration, error)`
- `plan:internal/shuttleengine#fakeEngine.KeepWarmCaps` -> `func (e *fakeEngine) KeepWarmCaps(role string) (time.Duration, time.Duration, error)`
- `plan:internal/shuttleengine/claudeengine#TestKeepWarmCaps` -> `func TestKeepWarmCaps(t *testing.T)`
- `plan:internal/shuttleengines#TestConfigModules_ClaudeCheckLoad` -> `func TestConfigModules_ClaudeCheckLoad(t *testing.T)`

**Uses:**
- `internal/shuttleengine/claudeengine#bashMaxTimeout`
- `internal/shuttleengine/claudeengine#resolvePromptCacheTTL`
- `internal/shuttleengine/claudeengine#loadConfigOrRefuse`
- `internal/shuttleengine/claudeengine#LoadConfig`

**Intent:** D4's cap becomes claude.yaml config beside the prompt-cache lifetime, answered through the provider seam (PATTERN-shuttle-provider-seam).

The `shuttleengine.Engine` interface gains the method `KeepWarmCaps(role string) (capWait, longCap time.Duration, err error)` beside `PromptCacheLifetime`: the longest one keep-warm wait of a session of that role blocks, and the long cap D5's long wait uses.
Its doc says a zero cap means the engine keeps nothing warm, so a caller blocks to its end, and an error means the engine's config cannot be resolved.
`role` is a base role; numbered roles are the resolver's business (card 3).

claude.yaml (`claudeengine.Config`) gains `KeepWarmCap string` (`keep_warm_cap`, template `250s`) and `KeepWarmCapRoles map[string]string` (`keep_warm_cap_roles`, template an empty map, `{}`), both `CategoryNone` in `ConfigSpec`, and `keep_warm_cap_roles` joins `ConfigOpenMaps`, so the map is the operator's own as `prompt_cache_ttl_roles` is.
Their template comments say: the keep-warm cap for a role without an entry, and per role keyed by a strand's role segment, replacing the default; values are Go durations, at least 60s, below the role's prompt-cache lifetime and at most 55m; they are validated when a run starts and on `lym config claude`; a cap above five minutes leaves that role's forks' caches cold, since forks run at Claude Code's 5-minute subagent lifetime.

`ValidateKeepWarm(claude Config) error` first resolves the prompt-cache TTLs as `resolvePromptCacheTTL` does, then checks the default cap against `prompt_cache_ttl` and, for every role named in either `prompt_cache_ttl_roles` or `keep_warm_cap_roles`, in sorted order, that role's cap (its `keep_warm_cap_roles` entry, else the default) against that role's lifetime.
Each cap must parse as a Go duration, be at least 60s, be below its lifetime and be at most `longKeepWarmCap` (55m: the one-hour Bash ceiling less five minutes); otherwise the error names the key (`keep_warm_cap` or `keep_warm_cap_roles[<role>]`), the value, the bound it broke and the way forward `fix it in claude.yaml with lym config claude` (PATTERN-refusal-way-forward).
`Claude.Prepare` calls it right after the TTL resolution and before any artifact is written, so a bad cap refuses a run start (the TTL error keeps precedence, since the TTLs are resolved first).
`Claude.KeepWarmCaps(role)` loads claude.yaml through `loadConfigOrRefuse`, runs `ValidateKeepWarm`, and answers the role's cap (its entry, else the default) and `longKeepWarmCap`; a load or validation error is returned.

The claude registry entry in `internal/shuttleengines`'s `registry` gets a `CheckLoad` that runs `claudeengine.LoadConfig` and then `claudeengine.ValidateKeepWarm` on the result, so `lym config claude --set` and an editor save refuse a cap the run start would refuse (configcli puts the file back when the load check fails).
The shuttleengines doc gains a sentence saying the claude module's load check also validates the keep-warm caps; the claudeengine doc's claude.yaml key list names the two new keys and states the cap rules and that `KeepWarmCaps` answers them with the long cap.

The fakes answer zero: `shuttlefake.Engine.KeepWarmCaps` answers `0, 0, nil` (doc: so no test-built gate or wait keeps a session warm, matching its `PromptCacheLifetime`), and so does `shuttleengine`'s internal `fakeEngine`.

Tests (tier0):
- `ttlConfig` also copies the template's `KeepWarmCap` and `KeepWarmCapRoles`, since `engineWith` marshals the whole Config and an empty cap would otherwise refuse every Prepare row.
- `TestLoadConfig`'s template row expects `KeepWarmCap: "250s"` and an empty `KeepWarmCapRoles`.
- `TestPrepare_RejectedBeforeArtifacts` gains a row whose claude.yaml sets `keep_warm_cap: 30s`, rejected before artifacts with an error naming `keep_warm_cap` and `lym config claude`.
- New `TestKeepWarmCaps` over `engineWith`: the template answers 250s and 55m for the roles `driver`, `webster`, `orch` and an unmapped role; a `keep_warm_cap_roles` entry `webster: 20m` answers 20m for `webster` and 250s for `driver`; refused, each naming its key and `lym config claude`: a default `59s`, a default `5m` against the 5m default lifetime, a `webster: 56m` entry above the long cap, a `driver: 6m` entry at or above driver's 5m lifetime, and a value `abc` that does not parse; an invalid claude.yaml (`engineWithInvalidConfig`) is the load error.
- New `TestConfigModules_ClaudeCheckLoad` in `internal/shuttleengines`: the claude module's `CheckLoad` passes over a temp anchor with no claude.yaml (the template) and refuses a local layer setting `keep_warm_cap: 10s`, naming `keep_warm_cap` and `lym config claude`.

**ImpactSummary:** one interface method added with its three implementers (claude, shuttlefake, the internal fake); claude.yaml gains two degrading CategoryNone keys and a validation that refuses a run start and a config write only on a cap the template never produces.
