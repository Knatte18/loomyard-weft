# Fixer-report, round 1 (code)

## Changes

All in `internal/shuttleengine/claudeengine/scope_test.go` unless noted. Commits `88a97ab4` and `309f1931`.

- **F5** (LOW), fixed as proposed: subtest `PrepareWithUnreadablePluginsFile` in `TestSessionScope_ReachesWrittenSettings` points `engine.claudeDir` at a malformed plugins file and asserts `disableClaudeAiConnectors` and `disableBundledSkills` are true and `enabledPlugins` is absent.
- **F6** (LOW), fixed as proposed: `TestClaudeConfigDir`, a non-parallel table test (comment says why) with a `CLAUDE_CONFIG_DIR` row and a `HOME` default row.
- **F7** (NIT), fixed as proposed: rows `FileUnderSkillsIsNotASkill` and `SeveralInstallsMergeTheirSkills` in `TestReadInstalledPlugins`.
  The empty-`installPath` case sits in its own non-parallel test, `TestReadInstalledPlugins_EmptyInstallPathIgnoresWorkingDirectory`, fixed differently from the proposal: as a table row it passed with the skip removed (see Self-check), so the test sets the working directory to one holding a `skills` directory with `t.Chdir`.
- **Parent decision, creel #790** (not a review finding, acted on as the parent's instruction): the decision record's "Interactive sessions are scoped too; only the orch keeps memory and ListAgents" entry replaces the interactive split.
  - `resolveSessionScope` now takes `orch bool` and `sessionScopeFor` takes the strand role, matched against `agentname.RoleOrch`.
  - Every session but the orch's gets `autoMemoryEnabled` false and `ListAgents` denied.
  - Every plugin that is on is limited to its preloaded skills, except `ly`, which keeps all of its skills.
  - `Prepare` and `OpenLaunch` pass `spec.Role`.
  - Updated `doc.go`, `template.yaml`, `docs/overview.md` and the `Prepare` doc comment to match.
  - Tests: the Orch row now turns the non-preloaded scribe skills off, new row `BoardSession`, `OpenLaunch` and new `OpenLaunchOrch` subtests assert the role split.
- Self-check LOWs: doc comments of `TestReadInstalledPlugins` and `TestSessionScope_ReachesWrittenSettings` updated, and `writeInstalledPlugins` now delegates to `writePluginsFile`.

- Gate repair: the gate flagged the two-line doc comment of `TestReadInstalledPlugins_EmptyInstallPathIgnoresWorkingDirectory` as fixed-column wrapped, so it is now one line, one sentence.

`lym gate test ./internal/shuttleengine/claudeengine` passed before the repair; the gate runs again after it.

## Disputed

Nothing disputed.

## Deferred with reason

F1–F4 are `decision`-class findings, which the fix step leaves out of this report.
The parent's recorded decisions (creel #790) accept F2, F3 and F4 as they stand, and F1's change is listed above.

## Self-check

- fix-code-correctness, LOW: `EmptyInstallPathHasNoSkills` passed with the skip removed. Fixed with a dedicated non-parallel test using `t.Chdir`.
- fix-consistency, LOW: `TestReadInstalledPlugins` doc comment stale. Fixed as proposed.
- fix-consistency, LOW: `TestSessionScope_ReachesWrittenSettings` doc comment omits the degrade subtest. Fixed as proposed.
- fix-consistency, LOW: `writeInstalledPlugins` twins `writePluginsFile`. Fixed by delegating.
- fix-proposal, LOW: duplicate of the helper finding above, merged.
- fix-proposal, LOW: duplicate of the stale doc comment finding above, merged.
- fix-affected-set: no findings.

Every fork returned a report.
