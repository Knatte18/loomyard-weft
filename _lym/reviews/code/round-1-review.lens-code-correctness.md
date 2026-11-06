```yaml
lens: code-correctness
outcome: reported
```

List: each function or branch, the case I ran, and whether it is correct.

- `resolveSessionScope`, scope off (`Enabled` false): returns the zero `sessionScope`. `omitempty` leaves out `disableWorkflows`, so a scope-off document has no new key. Correct.
- `resolveSessionScope`, scope on with nothing installed and nothing preloaded: `plugins` is empty, so `EnabledPlugins` stays nil. `SkillOverrides` is every `SkillsOff` entry. All three disable flags are true. Correct.
- `resolveSessionScope`, plugin-on rule:
  - `ly@loomyard` with `ly:board` preloaded: the namespace `ly` is found, so the plugin is on. Correct.
  - An id without `@`, such as `foo`: `Cut` returns the whole id as the name, and it is on exactly when a preloaded `foo:x` exists. Correct.
  - The `ly` plugin no longer has a special case, so it follows the same rule. Correct.
- `resolveSessionScope`, `SkillsOff` loop:
  - A preloaded skill is skipped. Correct.
  - A plugin skill that is not preloaded still gets an override, which does nothing and does no harm. The `ScopeConfig` comment says it stays listed. Correct.
- `resolveSessionScope`, orch and non-orch: a non-orch session gets auto-memory false and `ListAgents` denied, and the orch gets neither. Unchanged and correct.
- `sessionScopeFor`, scope off: returns the zero scope. Correct.
- `sessionScopeFor`, `installedPlugins` fails (unreadable directory or config directory not resolvable): the error is logged and `installed` is nil. The scope still carries the three flags and `SkillsOff`, and the doc comment says so. Correct.
- `readInstalledPlugins`, absent file: returns nil, nil. Correct.
- `readInstalledPlugins`, read error other than not-exist: returns a wrapped error. Correct.
- `readInstalledPlugins`, malformed JSON or `plugins` as an array: `Unmarshal` fails and the error is returned, so the malformed test still holds. Correct.
- `readInstalledPlugins`, `"plugins": null` or `{}`: returns an empty, non-nil slice. The caller only ranges over it, so this is correct.
- `readInstalledPlugins`, ids: they come back sorted, and the sorted-id contract holds.
- `readInstalledPlugins`, input tolerance (a behavior change, not a defect): the old typed decode (`map[string][]struct{...}`) failed on a plugin whose value was not an array. The new `json.RawMessage` decode accepts any value shape and lists the id. This is more tolerant and fits PATTERN-tolerant-records. Correct, and noted.
- `DisableWorkflows` reaching both paths: `sessionScope` is embedded in the run settings (`settings.go:221`, `buildSettings` at 383 and 405) and in the opened settings (`settings.go:190`, `buildOpenSettings` at 196 and 204). `Prepare` calls `sessionScopeFor` at `claudeengine.go:211` and `OpenLaunch` at `command.go:452`. The key is written in both paths, which settles the focus question about both paths.
- Removed code (`installedPlugin`, `lyPluginName`, `pluginSkillsDirName`): nothing in the non-test code under `internal` still references them. I confirmed this by grep on the symbols `scope.go` once held, which returned no hit outside `scope.go`. Correct.

No findings from this lens.

Tool calls:
1. grep of `sessionScope`, `sessionScopeFor`, `resolveSessionScope`, `readInstalledPlugins` and `installedPlugins()` callers in non-test `internal` Go files. My context lacked `Prepare`, `OpenLaunch` and the settings builders, which show that both paths embed the changed struct.
