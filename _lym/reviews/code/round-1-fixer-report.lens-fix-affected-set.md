```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set-premise searches for F5, F6 and F7 against the fix in `scope_test.go`. I made no tool calls, because my context already shows the changed file.

Unchecked findings: F1–F4 (decision class, not the fixer's to fix).

- **F7** (three untested `readInstalledPlugins` branches): all three now have a row in `TestReadInstalledPlugins`.
  - The empty `installPath` skip is covered by `EmptyInstallPathHasNoSkills`.
  - The non-directory entry skip is covered by `FileUnderSkillsIsNotASkill`.
  - The merge across installs is covered by `SeveralInstallsMergeTheirSkills`. Its `shared` skill sits in both installs and appears once in the want, so it exercises the dedupe as well as the merge.
  - The `os.ReadDir` error `continue` was not in the review's set and is not covered. That is outside the finding.
- **F6** (`claudeConfigDir`): both branches the review named are covered, the `CLAUDE_CONFIG_DIR` override and the `HOME` default.
  - The test is not parallel and says why in a comment.
  - It sets `HOME` as well, so the default row is deterministic.
  - The `claudeHomeDir` error branch is unreachable with `HOME` set, so omitting it is consistent with the review's "override row alone" fallback.
- **F5** (degrade path of `sessionScopeFor`): the new `PrepareWithUnreadablePluginsFile` row covers the `Prepare` path.
  - `OpenLaunch` shares `sessionScopeFor`, so the degrade branch is the same code. No second member of the set is left uncovered.
  - The row asserts `disableClaudeAiConnectors` and `disableBundledSkills` are true and `enabledPlugins` is absent, as the `fix` key asked.

No findings.

Calls made: none. The review, `scope.go` and the changed `scope_test.go` were already in context.
