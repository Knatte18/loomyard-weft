```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, name the test row that pins it.

**Per behaviour, the test that pins it**

1. **The verb exits 0 when `deny_python` is false and 1 when it is true, both at the hub's default layer, with nothing printed.**
   Pinned by `TestHookPythonAllowedVerb_FollowsTheHubsDenyPython`, rows PythonAllowed, PythonDenied and AllowedAgainAfterAChange (`internal/shuttlecli/hookpython_test.go:188-190`).
2. **The verb reads the value live, so a second write is seen.**
   Pinned by the same test, row AllowedAgainAfterAChange (`hookpython_test.go:190`).
3. **The verb exits 1 and reports on stderr when it cannot resolve a worktree.**
   Pinned by row OutsideAWorktree (`hookpython_test.go:191`).
4. **The verb exits 1 when the config does not load.**
   It shares its error branch with item 3 (`hookpython.go:61-63`), so item 3's row pins that branch.
   No row covers the config failure itself; acceptable.
5. **The verb skips the shuttle pre-run and resolves through `ResolveWithin`, so it answers from below the anchor or from the records sibling.**
   None: every row runs from the prime worktree's anchor or from outside any repo (`hookpython_test.go:195-198`).
6. **The Claude python hook runs `lym shuttle hook-python-allowed` from `$CLAUDE_PROJECT_DIR`.**
   Pinned by `TestShellGuardDenies_PythonRowAsksTheHubOnAMatch` (`hookcmd_integration_test.go:266-317`).
   - It denies unless lym exits 0.
   - It denies when lym is missing.
   - It never asks lym on a non-python command.
7. **The Codex python hook does the same, from its own working directory.**
   Pinned by `TestGuardCommand_PythonRowAsksTheHubOnAMatch` (`guards_run_test.go:765-825`).
8. **Claude installs the python row under every input.**
   Pinned by `TestBuildSettings_DenyToggleMatrix` (`settings_test.go:500`) and `TestBuildSettings_NoAgentHookAndForkGuard` (`settings_test.go:665`).
   Also by claude `TestPrepare_WritesTheManifest` and the settings golden.
9. **Codex installs the python row under every input.**
   Pinned by `TestInstalledGuards_CoverEveryRow`, row SwitchesOff (`guards_test.go:46`), and by codex `TestPrepare_WritesTheManifest` (`manifest_test.go:911`).
10. **Claude's notice names python only while `DenyPython` was true at launch.**
    Pinned by `TestBuildDenyNotice_MatchesInstalledDenies` (`settings_test.go:569`).
11. **Codex's notice names python only while `denyPython` was true at launch.**
    Pinned by `TestGuardEntries_RenderEveryRowWithItsCodexText`, through the `noticesWithoutPython` assertion (`guards_test.go:891`).
12. **Init panics on a switch missing from the announced map, in both engines.**
    None. That is a programmer-error guard, so no finding.

**Findings**

1. **LOW** · class `design` · `internal/shuttlecli/hookpython_test.go:188-191`
   - **Gap:** no row runs the verb from below the worktree's anchor or from the records sibling.
   - **What goes unpinned:** two behaviours the diff adds:
     - skipping the shuttle pre-run (`cli.go:81`);
     - resolving through `ResolveWithin`, documented as "anywhere under its anchored directory" (`hookpython.go:100-101`).
   - **Why it matters:** if either regressed to the strict `Resolve` gate, every row would still pass, because every row runs from the anchor or from outside any repo (`hookpython_test.go:195-198`).
   - **Fix:** add a row whose dir is a subdirectory of `hub.PrimeWorktree()` and expect exit 0 under `deny_python: false`.
2. **LOW** · class `consistency` · `internal/shuttleengine/claudeengine/settings_test.go`, `TestBuildSettings_DenyToggleMatrix`
   - **Report:** the redundancy report lists it as contained by `TestPrepare_WritesTheManifest`.
   - **What only it asserts:** the python, raw-go and publish rows present for every interactive, fork and publish combination, and the exact PreToolUse count of 7 (`settings_test.go:500, 511`).
   - **What the covering test asserts:** guard presence for one autonomous spec only (`manifest_test.go:132-142`).
   - **Fix:** mark it `//testtiming:keep pins each Bash deny's presence and the PreToolUse count across every interactive, fork and publish combination, which TestPrepare_WritesTheManifest checks for one spec`.
3. **LOW** · class `consistency` · `internal/shuttleengine/codexengine/guards_test.go:28`, `TestInstalledGuards_CoverEveryRow`
   - **Report:** the redundancy report lists it as contained by `TestPrepare_InstallsGuardsFromItsInputs`.
   - **Why the covering test cannot catch a drop:** it builds its expected guards and notice by calling `installedGuards` and `denyNotice` themselves (`prepare_test.go:362-367`). A wrongly dropped python row would therefore change the expectation along with the result.
   - **What only it asserts:** only the literal row-name lists catch that drop, the SwitchesOff row included (`guards_test.go:46`).
   - **Fix:** mark it `//testtiming:keep pins the literal installed row names per input, which TestPrepare_InstallsGuardsFromItsInputs derives from installedGuards itself`.

**Outside my lens:** nothing tests a standalone launch, whose config lives in the state dir. The verb resolves the target repo's layers instead (`standalonegeom/reedgeom.go:44-45`). That belongs to correctness and is not filed here.

Tool calls:
1. Read `codexengine/prepare_test.go:311-395`, the `TestPrepare_InstallsGuardsFromItsInputs` table, to judge the containment claim against its assertions. My context held only the diff hunks of that test.
