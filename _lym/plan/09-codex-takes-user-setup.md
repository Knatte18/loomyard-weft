# Card 9 — codex-takes-user-setup

**Create:**
- `plan:internal/shuttleengine/codexengine#Seams` -> `type Seams struct`
- `plan:internal/shuttleengine/codexengine#TestPrepare_UserSetup` -> `func TestPrepare_UserSetup(t *testing.T)`
- `plan:internal/shuttleengines#TestNew_CodexLaunchCarriesTheClaudeUserSetup` -> `func TestNew_CodexLaunchCarriesTheClaudeUserSetup(t *testing.T)`

**Edit:**
- `internal/shuttleengine/codexengine#NewFromConfig` -> `func NewFromConfig(cfg shuttleengine.Config, dirs configengine.Dirs, seams Seams) *Codex`
- `internal/shuttleengine/codexengine#Codex.readyHome` -> `func (c *Codex) readyHome(codex Config) (home string, writableRoots []string, found checkout, err error)`
- `internal/shuttleengine/codexengine#Codex`
- `internal/shuttleengine/codexengine#Codex.Prepare`
- `internal/shuttleengine/codexengine#Codex.OpenLaunch`
- `internal/shuttleengine/codexengine#Config`
- `internal/shuttleengine/codexengine#ConfigSpec`
- `internal/shuttleengine/codexengine#LoadConfig`
- `internal/shuttleengine/codexengine#newEngine`
- `internal/shuttleengine/codexengine#TestLoadConfig_Template`
- `plan:internal/shuttleengine/codexengine#TestLoadConfig_RefusesInvalidValues`
- `internal/shuttleengines#registry`

**Extends:**
- `internal/shuttleengine/codexengine/template.yaml`
- `internal/shuttleengine/codexengine/doc.go`
- `internal/shuttleengines/doc.go`
- `pattern/PATTERN-shuttle-provider-seam.md`

**Uses:**
- `plan:internal/shuttleengine#UserSetupSource`
- `plan:internal/shuttleengine#UserSetupRequest`
- `plan:internal/shuttleengine/claudeengine#NewUserSetupSource`
- `plan:internal/shuttleengine/claudeengine#RunPluginHook`
- `internal/shuttleengine/codexengine#repoFixture.addWorktree`
- `internal/shuttleengine/codexengine#warnLog`

**Intent:**
A Codex launch's developer instructions carry the user's Claude setup (D8), and Prepare's plugin hook runner and sandbox probe become injectable seams, as the record's Exports name.

`Seams` is `{UserSetup shuttleengine.UserSetupSource; RunHook shuttleengine.HookRunner; SandboxProbe func(bin, home, dir string) error}`.
`NewFromConfig` takes it: a nil `SandboxProbe` keeps `runSandboxProbe`, a nil `UserSetup` adds no user setup, and `RunHook` is handed to the source as given, so a nil runner lists each hook unrun.
`Codex` gains `userSetup` and `runHook` fields beside `sandboxProbe`.
The callers change with it: `shuttleengines`' codex `registry` constructor passes `Seams{UserSetup: claudeengine.NewUserSetupSource(dirs), RunHook: claudeengine.RunPluginHook}`,
and the test helper `newEngine` passes `Seams{SandboxProbe: …}` returning nil, the probe it sets today.
`New` in `new_test.go` builds the struct directly and is unchanged.
The other Prepare tests (`TestPrepare_ConcurrentLaunchesLoseNoRoot`, `TestPrepare_Warnings`, `TestPrepare_RefusesBeforeWriting`) go through `newEngine`, whose nil source adds nothing, so they are unchanged.

codex.yaml gains `plugin_hook_timeout`, template default `10s`, as `Config.PluginHookTimeout`, classified `CategoryNone`;
`LoadConfig` refuses a value that does not parse as a positive Go duration, naming the key.

`readyHome` also returns the checkout it read, so `Prepare` has the worktree root; `OpenLaunch` discards it.
After the sandbox settles, `Prepare` calls `userSetup.UserSetup` with `WorkDir` the engine's anchor, `RepoRoot` the checkout root, `Skills` `spec.Skills`, `Role` `spec.Role`,
`SessionID` the run's id (the run directory's base name, since no Codex session id exists yet), `HookTimeout` the parsed `plugin_hook_timeout` and `RunHook` the seam's runner.
A source error refuses the launch with the `codexengine:` prefix before `prompt.md` is written; the error carries the file and way forward the source names.
Each of the result's `Warnings`, and each `Omitted` hook with its cause, is logged through `c.warn`;
an omitted hook's warning names the way forward to fix the hook, or to turn its plugin off in the session scope as card 8's items say.
It appends the result's items to the developer instructions after the standing passages in D5's order:
the instruction files, then the skill listing as one item named `the skill listing`, then the plugin contexts.
Each keeps the name and way forward the source gave, so card 5's cap refusal names the largest file or plugin hook and says to shorten that file or turn its plugin off in the session scope.
The skill listing's way forward is to list fewer plugins in the session scope: drop plugin skills from the role's skills in agents.yaml (`lym config agents`) or the spec, or, while claude.yaml's `session_scope` is off, turn it on.

Tests:
- `TestPrepare_UserSetup` (tier0, parallel table over a fake source recording its request, with the engine's warnings in a `warnLog`):
  the request carries the anchor, the checkout root, the spec's skills and role, the run directory's base name and the 10s timeout;
  the developer instructions hold the creel head, then the files, the listing and the plugin context in that order;
  an omitted hook and a source warning each reach the warnings, the hook's naming its cause;
  a source error refuses with the source's way forward and no `prompt.md`;
  a global-file item over 128 KiB refuses naming its path and `way forward`, with no `prompt.md`.
  The nil `SandboxProbe` default is left to the production wiring: a row reaching it would run the Codex binary, which `newEngine`'s probe exists to keep out of every test.
- `TestLoadConfig_Template` expects `PluginHookTimeout: "10s"`; `TestLoadConfig_RefusesInvalidValues` gains rows for `plugin_hook_timeout: soon` and `plugin_hook_timeout: 0s`.
- `TestNew_CodexLaunchCarriesTheClaudeUserSetup` (tier0, not parallel since it sets `CLAUDE_CONFIG_DIR` and `HOME`) pins the wiring.
  It writes a worktree with a `.git` directory and `HEAD`;
  a codex.yaml local layer naming two temp homes, and the worktree layer `configengine.ConfigFile(anchor, "codex")` setting `sandbox_mode: danger-full-access` and `writable_roots: []`, since the local layer may not set the machine key, so no probe runs;
  an operator home holding `auth.json` and `sessions`; and a config directory holding a `CLAUDE.md`.
  It builds the set with `New`, runs the codex engine's `Prepare`, and asserts the launch line carries the global file's text.

`codexengine/doc.go` describes the seams and the user-setup items; `shuttleengines/doc.go` says the registry wires claudeengine's user-setup source and hook runner into the codex engine.
`PATTERN-shuttle-provider-seam.md` names the route: Claude's user setup reaches `codexengine` only through `shuttleengine.UserSetupSource`, wired in `internal/shuttleengines`.

**ImpactSummary:** `NewFromConfig` has two callers, the shuttleengines registry and the codex test helper, both on this card; `readyHome`'s two callers are in codexengine.go; production Codex launches gain the user-setup items.
