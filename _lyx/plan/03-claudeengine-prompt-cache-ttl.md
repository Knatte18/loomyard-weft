# Card 3 — claudeengine puts the role's prompt-cache TTL on both pane lines

**Edit:**
- `internal/shuttleengine/claudeengine/command.go`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine/doc.go`
- `docs/overview.md`
- `internal/shuttleengine/claudeengine/command_test.go`
- `internal/shuttleengine/claudeengine/prepare_test.go`
- `internal/shuttleengine/claudeengine/settings_test.go`
- `internal/shuttleengine/claudeengine/envfile_integration_test.go`
- `internal/webstercli/smoke_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#Config`

**Intent:**
Implement D1, D3, D4 and D5 inside `claudeengine`, the only package that names the env var or the `5m|1h` vocabulary (`PATTERN-shuttle-provider-seam`).

In `command.go`:
- A const `promptCacheTTLEnvKey = "CLAUDE_CODE_PROMPT_CACHE_TTL"` beside `forkSubagentEnvKey`, its comment saying it rides the pane line for the same reason, and that it outranks every Claude Code control but `FORCE_PROMPT_CACHING_5M`.
- The accepted values `5m` and `1h`, exact and case-sensitive, in one set like `validEfforts`.
- `resolvePromptCacheTTL(role string, cfg shuttleengine.Config) (string, error)`: it first validates `cfg.ClaudePromptCacheTTL` and every value in `cfg.ClaudePromptCacheTTLRoles`, walking the map in sorted key order so the reported entry is deterministic, then returns the map's entry for `role` when one exists and the default otherwise.
  An empty `role` takes the default; a map key naming no real role matches nothing (D3).
  A value outside the set, the empty string included, is an error naming the key (`claude_prompt_cache_ttl`, or `claude_prompt_cache_ttl_roles` and the role for a map entry), the offending value, the accepted values `5m` and `1h` (case-sensitive), and `shuttle.yaml` as the file to fix.
- `buildLaunchCmd` and `buildResumeCmd` each take a `promptCacheTTL string` parameter and wrap the line with `sh.WithEnv(promptCacheTTLEnvKey, promptCacheTTL, cmd)` after the fork-subagent wrap and before the `envFileKey` wrap, so `CLAUDE_ENV_FILE` still leads the line.
  The parameter is always non-empty, since `Prepare` validated it; each doc comment gains one sentence on the TTL assignment, and `buildResumeCmd`'s says the resumed claude inherits nothing from the launch, so it must carry the TTL too.

In `Prepare`:
- Resolve the TTL once with `resolvePromptCacheTTL(spec.Role, cfg)` among the existing validations, before any artifact is written, and pass the one value to both line builders, so the two lines agree by construction (D1).
- After resolving, log one `logger.Info` line through `internal/logger` carrying `role` and `ttl` (D5).
- The doc comment's validation sentence names the prompt-cache TTL config too.

What the refusal blocks and what bounds it: a bad value refuses `Prepare`, so it blocks only a new run, one adopting a session by `Spec.ResumeSessionID` included; a started or rebuilt session is untouched, since reed replays its stored resume line.
Because the whole configured set is checked, one bad entry refuses the next spawn of every role until `shuttle.yaml` is fixed, and the error names that file.
The refusal lets nothing through that was refused before: it only adds a guard.

Docs:
- `claudeengine/doc.go`: a paragraph beside the `CLAUDE_ENV_FILE` one stating that both lines carry `CLAUDE_CODE_PROMPT_CACHE_TTL`, resolved from `Spec.Role` through `shuttle.yaml`'s per-role map with the default as fallback, validated to `5m` or `1h` before any artifact, logged with the role, and fixed for the session's life since a resume line is never rebuilt; that the subagent bucket is left at Claude Code's default; and that `FORCE_PROMPT_CACHING_5M` in Claude Code's environment still wins.
- `docs/overview.md`: in the shuttle module entry, one sentence that the engine sets the prompt-cache TTL per role on launch and resume from `shuttle.yaml`'s `claude_prompt_cache_ttl` and `claude_prompt_cache_ttl_roles` (template: 1 hour for `driver` and `webster`, 5 minutes for every other role).

Tests:
- A test helper in `prepare_test.go` returns a `shuttleengine.Config` whose two TTL fields come from the shipped template, resolved through `shuttleengine.LoadConfig` on an empty temp directory, so fixtures follow the template rather than restating it.
  Every fixture that hands `Prepare` a `Config` takes its TTL fields from it: the `Config{}` and deny-only literals in `prepare_test.go`, `settings_test.go` (`TestPrepare_PromptLaunchLimit`, `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`) and `envfile_integration_test.go`, and in `internal/webstercli/smoke_test.go`'s `realForkSettingsPath` (an `llm` file in another package, which sets the two fields from `shuttleengine.LoadConfig` the same way).
- `command_test.go`: `TestBuildLaunchCmd`, `TestBuildResumeCmd` and `TestBuildResumeCmd_NoticePosix` pass a TTL; the per-dialect lead constants absorb the `5m` assignment after the `CLAUDE_ENV_FILE` one, so the existing exact-string rows change only through the constants, and one row per builder passes `1h` and asserts its exact line.
- `TestPrepare_RejectedBeforeArtifacts` gains a `cfg` column, the template config for the existing rows, and rows refusing an empty default (`Config{}`), `1H`, `5`, a map value left empty as YAML `null` unmarshals it, and a bad map entry for `orch` while `spec.Role` is `driver`.
  Each TTL row's `wantErrContains` names the key, plus the role for a map row; the rows also assert the error names `5m`, `1h` and `shuttle.yaml`, and the existing no-artifacts check covers them.
- `TestPrepare_ThreadsSpecIntoLaunchCmds` gives every row the template config and adds `onBothLines` rows asserting `CLAUDE_CODE_PROMPT_CACHE_TTL` with the quoted value on the launch and the resume line:
  role `driver` and role `webster` get `1h`, role `orch` and an empty role get `5m`;
  a map entry `orch: 1h` gives `orch` `1h`;
  a map without `driver` gives `driver` the default;
  a default of `1h` gives an unmapped role `1h`.
- A new `TestPrepare_LogsResolvedPromptCacheTTL` captures logger output with `logcapture.CaptureVerbose`, prepares a `driver` run under the template config, and asserts the captured line carries the `role` key with `driver` and the `ttl` key with `1h`, in the form the logger's handler renders attributes.
  It does not call `t.Parallel`, and a comment names the process-global logger as the reason (`PATTERN-test-speed`).
- `test-budget.yaml`: the `internal/shuttleengine/claudeengine` row rises from 89 to 90 for that one new test function, since `cmd/lyx/testbudget_test.go` fails a package over its row (`PATTERN-test-speed`).

**ImpactSummary:** both pane-line builders change signature and every launch and resume line gains one env assignment; Prepare now refuses a Config whose TTL fields are empty or invalid, so every test fixture handing Prepare a Config, across claudeengine and webstercli's llm smoke test, sets them from the template.

**Commit:** 3: claudeengine: set the prompt-cache TTL per role on the launch and resume lines
