```yaml
lens: code-correctness
outcome: reported
```

**Task:** lens code-correctness. For each changed function or branch, the case I ran and whether it came out correct.

## Cases run

- **`agentconfig.Load`**
  - Missing role in a hub file: the layer stack fills from the template, so no role starts with no list. Correct.
  - Role set to null, or a bare scalar: refused ("must be a list of strings"). Correct.
  - Role key outside `Roles`: refused. Correct.
  - `efforts` without `default`: refused. Correct.
  - `&lt;role&gt;/&lt;model&gt;` key with a bad role or empty model: refused. Correct.
  - Unreadable file: returned unwrapped. Correct.
- **`Config.Skills`**
  - Role list followed by `extra`, repeats dropped, fresh slice. Correct.
  - Zero `Config`: refused. Correct.
- **`Config.Effort`**
  - Precedence `role/model`, then `role`, then `model`, then `default`. Correct.
  - Empty role or model skips the keys that name it. Correct.
  - Zero `Config` returns "". Correct.
- **`Config.ModelEfforts`**
  - Role keys are not counted as models, and other roles' `&lt;role&gt;/&lt;model&gt;` keys are skipped.
  - Role "" yields model keys and `default`. Correct.
- **`shuttleengine` `start`**
  - `ConfigRole` empty leaves `Skills` untouched. Correct.
  - `ConfigRole` set puts the role list first and then `Spec.Skills` once each. Correct.
  - The merged list is the one the skill loader, the load step and `Prepare` receive. Correct.
  - A role with no list is refused before any strand is created. Correct.
  - Every production `NewRunner` call site takes a `shuttleCfg`. The `shuttleengine.Config{}` literals in `shuttleengines.go` only build engines, and `Prepare` receives the runner's config. Correct.
- **`LoadConfig`**
  - A broken `agents.yaml` fails every shuttle config load. That matches "refusal at load". Correct.
- **`claudeengine.Prepare`**
  - An explicit model-spec effort is kept. Correct.
  - Empty effort resolves to the role-and-model alias entry, else the role, else `default`. Correct.
  - `claude-haiku-5-5` maps to `haiku` and `claude-fable-5-1` to `fable`. Correct.
  - Bad table effort for the launch model fails `validateEffort` before any artifact is written. Correct.
- **`subagentEffortEnv`**
  - `scope.Env` is assigned after `sessionScopeFor`, so it is set even when `session_scope` is disabled (zero scope). Correct.
  - A role effort Claude Code does not accept is refused. Correct.
  - No effort returns nil, so no `env` key is written. Correct.
- **`subagentPassage`**
  - Prepended ahead of the standing passages. Correct.
  - Empty `skills` returns the effort text only. Correct.
  - `ConfigRole` "" (MultiLLM seats) gets the `default` effort. Correct.
- **`OpenLaunch`**
  - `effortFor(..., "", ...)` uses the model and `default` only. Correct.
  - It sets no `Env`, which is correct because an opened session has no role.
- **Orch**
  - `wireCLI` resolves `Skills(RoleOrch, nil)`. The watcher re-sends the same list the start path loads, so the reload and start lists are identical. Correct.
- **Gate fixer**
  - `gatefixer.SpawnOrAttach` sets `RoleGateFixer` itself. The verify fixer and the card fixer reach it with only the language skills. Correct.
- **`seatengine`**
  - `seatSpec` carries `ConfigRole`. Correct.
- **`burlerengine.seatSkills`**
  - Returns the language skills or nil. The `burler` role list comes first in shuttle. Correct.
- **Codex and non-Claude engines**
  - The shuttle skills merge is engine-neutral. The efforts table is applied only in `claudeengine`. Correct.

## Findings

1. **LOW, class `design`.** `internal/shuttleengine/claudeengine/subagents.go:28`: `modelAlias` takes the segment after `claude-` up to the next dash.
   - A legacy id such as `claude-3-5-sonnet-20241022` yields `3`, and a bracketed alias such as `opus[1m]` is kept whole, so neither matches the `opus` or `sonnet` entry.
   - Both fall through to `default`.
   - Effort for such a model is then not by model. This is harmless for the current 5-series ids, so I cannot tell from the diff whether such models are in use.

2. **LOW, class `design`.** `internal/agentconfig/config.go:197`: `checkEfforts` checks that values are non-empty but not that they are valid Claude efforts.
   - A model-keyed or role-keyed entry that the load or launch never validates (for example `haiku: hgih`) is validated only when a session launches on that model.
   - `subagentPassage` meanwhile tells every session to put that value in its Agent calls, and `subagentEffortEnv` validates only the role-level fallback.
   - A typo surfaces as a failing Agent call rather than a refusal at load.

## Tool calls

1. `grep` of `claudeengine/settings.go` and `scope.go` for scope and `env` handling. The manifest did not show how `buildSettings` carries `sessionScope.Env` or whether a disabled scope drops it.
2. `grep` of `shuttleengine/*.go` for readers of `Skills` and `ConfigRole`. The manifest did not show whether another entry point reads `Skills` outside `start`.
3. `grep` of production `NewRunner(` and `shuttleengine.Config` sources. The manifest did not show whether any production runner is built without `LoadConfig`'s `Agents`.
