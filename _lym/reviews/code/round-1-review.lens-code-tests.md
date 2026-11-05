```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**
- A plugin is on exactly when the session preloads a skill under its name: `TestSessionScope_WrittenSettingsPerRole`, rows Orch, OpenedSessionWithoutSkills and DarnWriter (`scope_test.go:79-97`).
- A disabled config writes no scope keys: ScopeOff row (`scope_test.go:98-102`), and the golden test in `settings_test.go` for a zero scope.
- `disableClaudeAiConnectors` and `disableBundledSkills` are true: all three enabled rows.
- A `skills_off` skill is off unless the session preloads it: the Orch row (`scribe:prose` is exempt) and the OpenedSessionWithoutSkills row (it is turned off).
- An autonomous session turns off the other skills of its plugins: DarnWriter row (`scribe:conversation` and `scribe:python-testing` are off).
- An interactive session keeps all skills of its plugins: Orch row (no per-skill overrides).
- `autoMemoryEnabled` is false and `ListAgents` is denied for autonomous sessions only: DarnWriter row for autonomous, Orch and Opened rows for absent, and the `Prepare` and `OpenLaunch` subtests of `TestSessionScope_ReachesWrittenSettings`.
- `readInstalledPlugins`: `TestReadInstalledPlugins` covers sorted ids, skills per plugin, a plugin with no skills directory (`bare@market`), an absent file and a malformed file.
- `Prepare` and `OpenLaunch` carry the scope into the written settings: the `Prepare` and `OpenLaunch` subtests of `TestSessionScope_ReachesWrittenSettings`.
- `claude.yaml` `session_scope` default from the template: `TestLoadConfig` (`config_test.go:61-78`).
- The reed board button's added `--skill` flags: `TestLoadConfig_TemplateDefaultsResolve` (`config_test.go:116`). This asserts the shipped config value rather than prose, so I raise no finding.
- `claudeConfigDir`, the `CLAUDE_CONFIG_DIR` and home-directory resolution: none.
- `sessionScopeFor` on a read error (log the error, name no plugin, still turn off connectors, bundled skills and `skills_off`): none.
- `readInstalledPlugins` branches for an empty `InstallPath`, non-directory entries under `skills/`, and several installs of one plugin merged: none.
- `ConfigSpec` category entry for `session_scope`: none seen. I did not check for a generic test over all keys.

**Findings**

1. LOW, class `scope`, `internal/shuttleengine/claudeengine/scope.go:136-145`. The degrade path of `sessionScopeFor` has no test. If it returned the zero scope on error, the connectors and bundled skills would silently stay on. Every test either sets `engine.claudeDir` to a valid directory or calls `resolveSessionScope` directly. Evidence: `scope_test.go:162-211` and `prepare_test.go` `engineWithAnchor` set a clean `claudeDir`; no test pairs a malformed plugins file with `Prepare` or `OpenLaunch`. Fix: add one `Prepare` row with `claudeDir` holding a malformed plugins file, asserting `disableClaudeAiConnectors` and `disableBundledSkills` are true and `enabledPlugins` is absent.

2. LOW, class `scope`, `scope.go:160-170`. `claudeConfigDir` has no test, so a `CLAUDE_CONFIG_DIR` set on the machine and `~/.claude` under the home directory are both unpinned. Every test bypasses it through `engine.claudeDir`. Fix: add a table test for `claudeConfigDir` that sets the environment variable. It must be non-parallel, with a stated reason, since `t.Setenv` is used.

3. NIT, class `scope`, `scope.go:203-214`. `readInstalledPlugins` branches for an empty `installPath`, a non-directory entry under `skills/`, and two installs of one plugin are untested. These are minor and could be rows in `TestReadInstalledPlugins`.

I found no duplicated test and no assertion on static prose.

**Tool calls:** none. The inherited context already held every file I needed.
