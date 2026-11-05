```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: one finding, and every other added row and subtest traced correct.

**Findings**

- **LOW** — `TestReadInstalledPlugins`, row `EmptyInstallPathHasNoSkills` (`scope_test.go`): this row passes with the defect in place, so it pins nothing.
  - With the `install.InstallPath == ""` skip removed, `readInstalledPlugins` calls `os.ReadDir(filepath.Join("", "skills"))`, which is the relative path `skills`.
  - The test's working directory is the package directory, and it has no `skills` directory (`ls` above), so `ReadDir` errors and the `continue` on that error hides the missing skip.
  - The row still yields `{id: "p@m"}` and passes.
  - It discriminates only if the test runs where a `skills` directory exists in the working directory.
  - A fix would run the row with the working directory set to a directory holding `skills/<x>`. That needs `t.Chdir`, which makes the test non-parallel.
  - Another option is to drop the row's claim that it pins the skip and leave it as a documented non-discriminating case.

**Traced correct**

- **`FileUnderSkillsIsNotASkill`:** the install holds dir `real` and file `notes.md`. The expected result is `["p:real"]`. Without the `IsDir` check, `p:notes.md` appears and the row fails.
- **`SeveralInstallsMergeTheirSkills`:** the `skills` set gets `shared`, `first` and `second`, and sorting gives `p:first`, `p:second`, `p:shared`, which matches `want`. An overwrite instead of a merge drops `first` or `second`, so the row fails.
- **`TestClaudeConfigDir`:**
  - The parent test is not parallel, so `t.Setenv` in the subtests is legal. The comment explains why.
  - `EnvironmentOverride` returns the override. If the override branch is removed, the result becomes `home/.claude` and the row fails.
  - `HomeDefault` sets `CLAUDE_CONFIG_DIR=""`, which `os.Getenv` reads as unset. `claudeHomeDir` honors `HOME`, so the result is `home/.claude`.
- **`PrepareWithUnreadablePluginsFile`:**
  - A malformed plugins file makes `installedPlugins` return an error, which `sessionScopeFor` logs.
  - `resolveSessionScope` then gets no plugins, so `enabledPlugins` stays nil and is omitted, while the connectors and bundled-skills flags are true.
  - A change that returned the zero scope on error would drop those two keys and fail the `got[key] != true` check.
  - A JSON-decoded `true` compared against the untyped constant `true` compares correctly.
- **Callers:** no production code or signature changed. `writeMalformedPlugins` replaced the inline setup in `TestReadInstalledPlugins` with the same behavior.

**Calls made**

1. `ls -d …/claudeengine/skills`: my context did not show whether the package directory holds a `skills` directory, which decides whether the empty-`installPath` row discriminates.
