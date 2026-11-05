```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I checked each function and branch the diff adds or changes by tracing a concrete input through it. I made no tool calls and used only the inherited context.

**Case list**

- `resolveSessionScope`, scope off (`Enabled` false): returns the zero scope, so no keys are written. Correct.
- `resolveSessionScope`, interactive orch preloading `scribe:*` and `ly:*`:
  - `ly` and `scribe` come out on, and every other installed plugin off.
  - No per-skill overrides come from plugins, because the interactive path `continue`s.
  - `SkillsOff` entries not preloaded are off. Memory and `ListAgents` are left alone. Correct.
- `resolveSessionScope`, autonomous Go writer preloading `scribe:prose` and `scribe:golang-testing`: `scribe` on, and every other `scribe` skill (python, conversation) off through `skillOverrides`. Memory is `false` and `permissions.deny` is `["ListAgents"]`. Correct.
- `resolveSessionScope`, no preloaded skills, interactive: all plugins off, `SkillsOff` entries all off, connectors and bundled skills off. Correct.
- `resolveSessionScope`, `installed` nil or empty (config dir unreadable): `EnabledPlugins` stays nil and is omitted, so the user's and the repo's plugin settings decide. The code and the docs both state this. Consistent, but see the failure-path finding below.
- `resolveSessionScope`, preloaded skill with no colon, plugin id with no `@`: the first is skipped when building namespaces, and the second uses the whole id as its name. No panic and no wrong result.
- `sessionScopeFor`, disabled config: returns the zero scope without reading the plugins file. Correct.
- `sessionScopeFor`, `installedPlugins` error: logged with `logger.Warn`, then the scope is still resolved from nil, so the connectors, bundled skills and `SkillsOff` stay off. Not swallowed silently. It matches the documented behavior.
- `installedPlugins`, empty `claudeDir`: falls back to `claudeConfigDir()`, which uses `CLAUDE_CONFIG_DIR` if set and otherwise `~/.claude`. Correct.
- `readInstalledPlugins`:
  - An absent file returns `nil, nil`. Correct.
  - A read error returns a wrapped error. Correct.
  - `"{"` returns a wrapped parse error. Correct.
  - A plugin with an empty or unreadable `installPath` gets an empty skill list. Silent, but documented.
  - Duplicate install entries, user and project scope, merge through the `skills` map and are sorted. Correct.
  - A plugin with no install entries still gets an entry in `plugins`, so it is switched off. Correct.
- `buildSettings` and `buildOpenSettings` with the embedded unexported `sessionScope`: `encoding/json` promotes its exported fields, `omitempty` drops a zero scope, and the test pins the exact keys. A zero scope leaves the document unchanged. Correct.
- `Prepare`: the scope is built after all validation and before the write, and a failed plugin read does not fail the launch. `Interactive` and `Skills` come from the spec. Correct.
- `OpenLaunch`, in a hub (`creelFlagDirForHook != ""`): the scope is written into `settings.json`, and the resume line reuses the same file. Correct. Outside a hub no settings are written and no scope applies, as documented. The Goal says "every session", so this is a gap, but it is a design question and not a code error.
- `writeOpenFiles`: the scope is passed through to `buildOpenSettings`, and the failure path still removes `dir`. Correct.

**Findings**

1. **LOW, design**, `internal/shuttleengine/claudeengine/scope.go:210-213`. `readInstalledPlugins` uses `entry.IsDir()` on `os.ReadDir` entries. A skill directory that is a symlink to a directory reports `IsDir()==false`, so the skill is missing from `plugin.skills`. An autonomous run then never gets an override turning that skill off, and the over-loading goes unnoticed. Fix: stat the entry, for example with `os.Stat(filepath.Join(dir, entry.Name()))`, or accept `entry.Type()&amp;fs.ModeSymlink != 0`.

2. **LOW, design**, `internal/shuttleengine/claudeengine/scope.go:206-209`. The `os.ReadDir` error on a plugin's `skills` directory is dropped with `continue`, with no log. A permission error or a moved install path gives a plugin with no skills, and an autonomous run silently keeps all of that plugin's skills. Fix: `logger.Warn` any error that is not `fs.ErrNotExist`.

3. **LOW, design**, `internal/shuttleengine/claudeengine/scope.go:140-144`. When the plugins file is unreadable, `EnabledPlugins` is omitted entirely. In an autonomous run, plugins such as `ly@loomyard` then stay as the user's settings and the repo's `.claude/settings.json` say. This is the failure the task is meant to prevent, so the fail-open here contradicts "everything else is off for that session, whatever the user settings say". It is logged and documented, but there is no fail-closed alternative, such as refusing the spawn or disabling the known plugin ids. Decide whether to keep the fail-open.

Everything else on the case list is correct.

Tool calls: none.
