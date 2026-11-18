```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, name the test row that would fail if it broke.

**Per behaviour, the test that pins it**
1. The verb exits 0 or 1 by the hub's `deny_python` at the default layer, re-read on every call: `TestHookPythonAllowedVerb_FollowsTheHubsDenyPython`, rows PythonAllowed, PythonDenied and AllowedAgainAfterAChange.
2. The verb resolves from a cwd below the anchor: same test, row AllowedFromBelowTheAnchor.
3. The verb exits 1 with a JSON error when no worktree resolves and no state dir exists: same test, row OutsideAWorktree.
4. The verb prints nothing on stdout when it allows or denies: same test (`out.Len() != 0` check) and `TestHookPythonAllowedVerb_FollowsAStandaloneRunsStateDirConfig`.
5. The verb skips the shuttle group's shared pre-run: pinned indirectly. The standalone test runs from a non-repo dir and expects exit 0, which the pre-run's geometry resolution would refuse.
6. Standalone fallback to the state dir derived for the target: `TestHookPythonAllowedVerb_FollowsAStandaloneRunsStateDirConfig`, but only for a target that is not a git repo (finding T1).
7. A resolved worktree with no Board and no state dir falls back to the worktree's own layers (`hookpython.go:94`): none (T2).
8. A config that fails to load gives exit 1 with a JSON error: none (T3).
9. The verb skips the root stencil seed (`SkipStencilSeedAnnotation`): none for this verb (T4).
10. The Claude hook runs `lym shuttle hook-python-allowed` from `$CLAUDE_PROJECT_DIR` and denies unless it exits 0; with no lym on PATH it denies; a non-python command never runs lym: `TestShellGuardDenies_PythonRowAsksTheHubOnAMatch`, all four rows.
11. The Claude hook denies when the project dir cannot be entered: none (T5).
12. The Codex hook asks lym from its own cwd, with the same allow, deny, no-lym and non-match behaviour: `TestGuardCommand_PythonRowAsksTheHubOnAMatch`.
13. The python row is installed under every input:
    - Claude: `TestBuildSettings_DenyToggleMatrix` (`!pythonInstalled`), `TestBuildSettings_NoAgentHookAndForkGuard` (exactly one python entry), and the PreToolUse counts in `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`.
    - Codex: `TestInstalledGuards_CoverEveryRow`, row SwitchesOff.
    - Both manifests: `TestPrepare_WritesTheManifest`, claude and codex.
14. The notice names python only while `DenyPython` is true at launch:
    - Claude: `TestBuildDenyNotice_MatchesInstalledDenies`.
    - Codex: `TestGuardEntries_RenderEveryRowWithItsCodexText`, the "with python allowed" assertion.
    - Codex `Prepare` threads `guardIn` into the notice: `TestPrepare_InstallsGuardsFromItsInputs` (`wantCreelHead(guards, tt.wantInputs)`).
15. The orch launch wiring carries the default-layer `deny_python` into the notice and keeps the row: `TestOrchIntegration_WireCLILaunchFollowsDenyPythonAtTheDefaultLayer`, rows DenyPythonFalse and DenyPythonTrue.
16. The Claude `buildDenyNotice` falls back to `installed` when `announced` is nil: `TestBuildDenyNotice_MatchesInstalledDenies`, through the claude-only denies.
17. The init-time panic in claudeengine and codexengine for a switch neither switch map knows: none. This is a programmer-error guard and needs no test.
18. Doc, template and help prose: exempt.

**Findings**

- **T1, MEDIUM, design.** `internal/shuttlecli/hookpython_standalone_test.go:343`. The standalone test misses the real standalone path. Its target is a bare `t.TempDir()` with no `.git`, so `lymcwd.ResolveWithin` fails and the verb takes the error branch.
  - A real standalone target is a git repo. There `ResolveWithin` succeeds with `hubDirs.Board == ""` (`hookpython.go:81-87`), and the cwd may sit below the target, so `RepositoryRootOf` has to climb.
  - The expected dir comes from `cliwire.StandaloneStateDir`, the function under test, not from the launch's `ResolveStandalone(...).StateDir` (`internal/cliwire/standalone.go:84-89`). So nothing pins that the verb and the launch agree on the dir.
  - A change that returned `hubDirs, nil` whenever `ResolveWithin` succeeded would pass this test.
  - Fix: add a row whose target is a git repo, run once from a subdirectory, and take the expected state dir from the standalone launch's own resolution.
- **T2, LOW, design.** `internal/shuttlecli/hookpython.go:94`. The branch where a worktree resolves with no Board and no standalone state dir returns the worktree's own layers, and no row covers it.
- **T3, LOW, design.** `internal/shuttlecli/hookpython.go:88-91`. The load-failure path has no row: a config that does not load should give exit 1 and a `shuttle: ` JSON error. Only the resolve-failure error is pinned, by row OutsideAWorktree at `hookpython_test.go:455`.
- **T4, LOW, design.** `internal/shuttlecli/hookpython.go:250` (the annotation map). Nothing pins that `hook-python-allowed` declines the root stencil seed, a behaviour the fix commit `47d84ce3` claims. `cmd/lym/stencilseedgate_test.go:36` checks the annotation's meaning on synthetic commands, not that this verb carries it.
- **T5, NIT, design.** `internal/shuttleengine/claudeengine/settings.go:339`. The doc claims a project dir lym cannot enter keeps the deny, and `TestShellGuardDenies_PythonRowAsksTheHubOnAMatch` (`hookcmd_integration_test.go:543-546`) has no row for it.
- **T6, NIT, consistency.** `internal/shuttleengine/claudeengine/settings_test.go:780-782`. In `TestBuildSettings_DenyToggleMatrix` the `pythonDeny` column no longer drives any assertion, since line 789 asserts the row unconditionally. The python_only rows now differ from the base rows only in a field nothing checks; they could be dropped or folded.

**Tool calls**
1. Read `cliwire.ResolveStandalone` and searched for `SkipStencilSeedAnnotation` tests and other `StandaloneStateDir` callers. My context did not hold the launch's state-dir derivation or the seed-skip tests.
2. Read the test names in `stencilseedgate_test.go` and `resolveStandaloneTarget`. I needed to know whether the seed-gate test covers real commands and how the launch normalizes its target.
