```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals for F5, F6 and F7, which the fixer carried out as proposed, in `scope_test.go`.

No fix-proposal case turns the proposals wrong. I checked each against a concrete case.

- **F5, `PrepareWithUnreadablePluginsFile`:** The test fails with the defect in place. If `sessionScopeFor` returned the zero scope on error, `got["disableClaudeAiConnectors"]` would be nil rather than `true`. The `enabledPlugins`-absent assertion also holds, because `installed` is nil, `plugins` is empty and `len(plugins) > 0` is false.
- **F6, `TestClaudeConfigDir`:**
  - `t.Setenv` sits only in subtests under a non-parallel parent, which Go requires.
  - The `HomeDefault` row sets `CLAUDE_CONFIG_DIR` to `""`, which `claudeConfigDir` reads as unset, so the row tests the home default.
  - `claudeHomeDir` reads `HOME` first, so the `HOME` redirect works.
- **F7:**
  - **Empty install path:** `EmptyInstallPathHasNoSkills` yields `{id: "p@m"}` with nil skills, matching the want.
  - **File entry:** `FileUnderSkillsIsNotASkill` yields only `p:real`.
  - **Several installs:** `SeveralInstallsMergeTheirSkills` pins the dedup of `shared` and the sorted merge.
  - Each row fails if its branch is removed.

Two small findings:

1. **LOW** — `writePluginsFile` in `scope_test.go` repeats the marshal, `MkdirAll` and `WriteFile` tail of the existing `writeInstalledPlugins`.
   - Evidence: both helpers end with the same four steps, writing `{"version":2,"plugins":...}` to `installedPluginsPath`. The fix added the second copy instead of having `writeInstalledPlugins` call it.
2. **LOW** — The doc comment of `TestReadInstalledPlugins` is now incomplete.
   - Evidence: it says the test pins "each plugin with the skill directories of its install path, none for an absent file, and an error for a malformed one". The fix added rows for an empty install path, a file under `skills` and several installs that the comment does not mention.

Tool calls: none; the context held every changed file.
