```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change I list the set I found and whether it is covered.

**Per-change list**

1. **`buildSettings` gained a `scope` parameter.** Every caller passes it. `claudeengine.go:212` and `scope_test.go:108` pass a real scope. All other test callers (`settings_test.go`, `hookcmd_integration_test.go`) pass `sessionScope{}`. Covered.
2. **`buildOpenSettings` and `writeOpenFiles` gained a `scope` parameter.** The only chain is `command.go:453` → `command.go:481`. Covered.
3. **The board button command now preloads `scribe:prose` and `scribe:conversation`.** The command string sits in `template_posix.yaml:70`, `template_windows.yaml:70` and `config_test.go:116`, and all three are updated. `docs/overview.md:379` describes the board agent but does not name its skills, so it stays correct. Covered.
4. **New `session_scope` key in `claude.yaml`.**
   - It is registered in `ConfigSpec` as `CategoryNone` (`config.go:67`).
   - It is in the template and in the `LoadConfig` expectation in `config_test.go`.
   - It is in the `doc.go:65` key list.
   - `docs/overview.md` never lists `claude.yaml` keys. Its only `claude.yaml` mentions are at line 435, and it has no `batching_nudge` entry.
   - Two other test files mention `keep_warm_cap_roles` and `batching_nudge`: `shuttleengines/keepwarm_test.go` and `loomcli/wiring_test.go`. They read other keys, not the full key set, so I found nothing there that needs updating.
   - The new key is new, not a changed meaning, so `PATTERN-config-layers` needs no `hubreconcile` reinterpretation. Covered.
5. **Docs: `doc.go` and the `docs/overview.md` shuttle row.**
   - `doc.go` states the scope behaviour. It describes `OpenLaunch` and `Prepare`.
   - It says the scope reaches Claude settings via `--settings`.
   - Its "Bound:" line says no launch has shown that `skillOverrides` works.
   - The shuttle row of the overview was updated.
   - No `PATTERN.md` or `pattern/` entry mentions settings, `enabledPlugins` or `installed_plugins`.
   - The diff touches neither `PATTERN.md` nor `pattern/`.
   - No declared PATTERN change exists for the fork to check. Covered.
6. **Commit messages.** I found no `Pattern-Departure:` trailer in any of the three commits.
   - **Commit message vs. the `ListAgents` tool.** The commit and `summary.md` put `ListAgents` in `permissions.deny`, which `scope.go:129` and `command.go` bear out. The Goal says `--disallowedTools`.
   - **The `ListAgents` departure is outside this lens.** `summary.md:27` states the departure. The handler and the correctness lens should judge whether it is licensed.
7. **Other engines.** `codexengine` does not read the scope, and I found no reference to it. The Goal names claudeengine only. Covered.

**Findings**

1. **LOW, consistency, `docs/overview.md:435`.** The shuttle row does not mention the new `session_scope` behaviour for spawned roles or opened sessions beyond what the diff added. I read only a truncated slice of the changed line, so I cannot confirm this one. It is unverified and not a firm finding.

I found no other member of the affected set that the diff leaves wrong.

**Tool calls (4)**
1. Grep of `buildSettings(`, `buildOpenSettings(` and `writeOpenFiles(` callers. My context had only the changed files, not every caller.
2. Grep for the board button command string across the repo. This found any other copy of the old command.
3. Grep for files listing `batching_nudge` or `keep_warm_cap_roles`. This found docs and fixtures that enumerate `claude.yaml` keys.
4. Grep of `pattern/`, `PATTERN.md` and `docs/*.md` for settings and plugin terms. This checked whether any rule document describes the changed behaviour.
5. Grep of `docs/overview.md` and the reed docs for `claude.yaml` keys and the board button. Neither the overview nor the reed docs name the button's skills or list `claude.yaml` keys.
