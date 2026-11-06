```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. List per behaviour, the test that pins it, or none.

**Per-behaviour list**

- `disableWorkflows` written in every scoped settings document, for the orch, a non-orch run and an opened session. Pinned by `TestSessionScope_WrittenSettingsPerRole` (`scope_test.go:83-103`). Every scoped row carries `"disableWorkflows":true` in exact-equality JSON, and the keys list at line 17 includes it, so dropping the field fails the test. Covered.
- `disableWorkflows` on the Prepare path with a plugins file that cannot be read. Pinned by `PrepareWithUnreadablePluginsFile` (`scope_test.go:234`). Covered.
- `disableWorkflows` on the `Prepare` path through the real `sessionScopeFor` wiring, and on the `OpenLaunch` path. The readable-plugins subtest `Prepare` asserts only `enabledPlugins` and `autoMemoryEnabled`. Both `OpenLaunch` subtests assert only `enabledPlugins`, `autoMemoryEnabled` and `permissions`. The unreadable-file subtest does cover the `disableWorkflows` write in `Prepare`, so `Prepare` is pinned. `OpenLaunch` is not: no test asserts `disableWorkflows` on a settings file it wrote. If `OpenLaunch` dropped the scope's Workflow field, no test row would fail. The sole assertion of the key in this lens's evidence is on `Prepare`. See finding F1.
- Zero scope with the scope off emits no `disableWorkflows`. Pinned by the `ScopeOff` row (`scope_test.go:105-109`, `want {}`). Covered.
- Plugin skills no longer get `skillOverrides`. This covers the `ly` exception being gone and a plugin that is on listing all its skills. Pinned by the `Orch`, `BoardSession` and `DarnWriter` rows (`scope_test.go:83-103`). Their `skillOverrides` hold only `anthropic-skills:docx`, and `scribe:golang-testing` and `scribe:python-testing` are absent. Exact equality would fail if plugin-derived overrides came back. Covered.
- `skills_off` still applies, minus the skills the session preloads. Pinned by the `OpenedSessionWithoutSkills` row, where `scribe:prose` is off, and by the `DarnWriter` row, where `scribe:prose` is preloaded and so absent. Covered.
- The plugin-on rule, including ids without `@` and the `ly` plugin without a special case. The `ly` plugin being on through `ly:board` is pinned by the `Orch` and `BoardSession` rows. An id without `@` is covered by none: `strings.Cut(id, "@")` on a bare name is never exercised, since every fixture id has `@`. This case existed before the diff too, so it is not a regression. Not a finding.
- `readInstalledPlugins` returns sorted ids only. Pinned by `NamesEachPluginSorted` (`scope_test.go:141-145`). The fixture lists `scribe@scribe` before `bare@market`, so the sort is exercised. Covered.
- `readInstalledPlugins` returns none for an absent file. Pinned by `AbsentFileNamesNone`. Covered.
- `readInstalledPlugins` returns an error for a malformed file. Pinned by `MalformedFileIsAnError`. Covered.
- A `plugins` value that is not an object, such as an array. This is new, because the decode target changed from a map of structs to `map[string]json.RawMessage`. `writeMalformedPlugins` (its body is not shown here) most likely writes invalid JSON. No test pins a wrong-shaped but valid JSON `plugins` value. That is a type change that behaves the same on every plugin value shape, so it is not a behaviour change worth a row. Not a finding.
- Removed coverage. The deleted tests pinned merging installs, a plain file under `skills`, an empty install path against the working directory, and the sorted skills. All of that code is deleted, so no behaviour that still exists lost its only test.
- Test economy of the new rows. `NamesEachPluginSorted` replaces three deleted rows, and no new test duplicates an existing one. The `PrepareWithUnreadablePluginsFile` change extends an existing row instead of adding a test. No redundancy.

**Findings**

- F1. Severity LOW, class `scope`. Location: `internal/shuttleengine/claudeengine/scope_test.go:244-278`, the `OpenLaunch` and `OpenLaunchOrch` subtests. The `OpenLaunch` path (the opened-session path, via `sessionScopeFor`) has no assertion of `disableWorkflows`, `disableBundledSkills` or `disableClaudeAiConnectors`. The diff adds the key to the scope struct and the `OpenLaunch` scope write, so a regression that drops it on that path would pass. Evidence: `scope_test.go:267-276` asserts only `enabledPlugins`, `autoMemoryEnabled` and `permissions`. Fix: in the `OpenLaunch` subtests, also assert `got["disableWorkflows"] == true`.

I made one tool call.
- Read `scope_test.go:100-280`. The context held only the diff, not the `ScopeOff` row, the Prepare subtests or the `OpenLaunch` subtests.
