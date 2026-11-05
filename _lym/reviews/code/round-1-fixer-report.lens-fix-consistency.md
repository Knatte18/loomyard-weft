```yaml
lens: fix-consistency
outcome: reported
```

Task: check the `scope_test.go` change set (F5, F6, F7) for agreement with its surroundings.

No BLOCKING or MEDIUM findings.

- **LOW** — `TestReadInstalledPlugins` doc comment, `scope_test.go`.
  - The comment says the test pins "each plugin with the skill directories of its install path, none for an absent file, and an error for a malformed one".
  - The fix added rows for an empty `installPath`, a plain file under `skills`, and several installs merging their skills.
  - The comment now omits what those three rows pin.

- **LOW** — `TestSessionScope_ReachesWrittenSettings` doc comment, `scope_test.go`.
  - The comment describes only the scope a run and an opened session get when the plugins file is readable.
  - The new `PrepareWithUnreadablePluginsFile` subtest pins the degrade path: connectors and bundled skills off, `enabledPlugins` absent.
  - The comment does not mention it.

- **LOW** — `writeInstalledPlugins` and the new `writePluginsFile`, `scope_test.go`.
  - Both build a config directory, marshal `{"version":2,"plugins":...}` and write `installedPluginsPath`, with the same marshal and write steps.
  - `writeInstalledPlugins` only adds the creation of skill directories before that.
  - It could build its install paths and delegate to `writePluginsFile`. The fix left the twin as it was.

- No finding: the file header still fits, since `claudeConfigDir` falls under "reading of Claude Code's installed plugins" closely enough.
- No finding: `TestClaudeConfigDir` sets `HOME`, which matches how `claudeHomeDir` resolves the home directory (`audit.go:112`), and it is non-parallel with the reason stated in a comment.
- No finding: the new `claudeConfigDir` test names (`EnvironmentOverride`, `HomeDefault`) follow the file's `Scenario` convention.

Further tool calls: none. My context already held every changed file.
