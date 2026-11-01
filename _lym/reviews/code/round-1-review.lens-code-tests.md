```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: per behaviour, the test that pins it, or none**

1. Headless-claude pattern denies `-p` / `--print` forms and passes the rest.
   - Pinned by `TestRows_PatternsDenyAndAllow` (`internal/shellguard/rows_integration_test.go` diff, about lines 210–222).
   - Deny rows: short flag, long flag, `--print=json`, a flag after other arguments, a path-prefixed binary, `claude.exe`, and a launch after `&&`.
   - Allow rows: interactive launch, `claude` as an argument of another command, a quoted mention, `claudette -p`, and a later command's `-p`.
   - The `x; claude -p hi` row sits in the second table.
   - The documented guardrail gaps (`bash -c`, `env`, an assignment prefix) have no rows. The row comment states they pass, so this is not a finding.
   - A launch after a pipe and `sudo claude -p` have no row. `sudo` passes by the row's own comment, and the `&&` and `;` rows already exercise the same separator alternation.
   - Combined short flags such as `-pd` have no row. The pattern requires a whole-word `-p`, so such a form would not be matched and nothing pins either result.
2. The row is installed in every run mode with no switch.
   - Pinned by `TestBuildSettings_DenyToggleMatrix` (`headlessClaudeInstalled` under every input) and by `TestBuildSettings_NoAgentHookAndForkGuard` (exactly one headless-claude entry).
   - The matrix and `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` pin the PreToolUse counts (5 base entries, 6 with `refuse_raw`).
3. The notice announces the headless-claude sentence.
   - Pinned by `TestBuildDenyNotice_SentenceContent` (the loop over `computed-rm`, `machine-sleep` and `headless-claude` for autonomous, interactive and fork runs) and by `TestBuildDenyNotice_MatchesInstalledDenies`.
4. No Agent hook under any input, and the notice has no Agent sentence.
   - Pinned by `TestBuildSettings_NoAgentHookAndForkGuard` and `TestBuildSettings_DenyToggleMatrix` (`hasMatcher("Agent")`).
   - The golden file (`TestBuildSettings_MatchesGolden`, now 16 combinations) also pins it.
5. The Webster fork guard stays.
   - Pinned by `TestBuildSettings_NoAgentHookAndForkGuard`, rows `fork_on_installs_the_bash_guard`, `fork_off_installs_no_bash_guard` and `fork_with_ask_deny`.
6. A repo default still holding `deny_agent_tool` loads without it.
   - Pinned by the `config_test.go` subtest "a repo default holding the retired deny_agent_tool loads without it".
7. A local override setting `deny_agent_tool` is refused, naming `claude.local.yaml` and the key.
   - Pinned by the `config_test.go` loop row `deny_agent_tool: false`.
   - The way-forward text (`retiredAgentDenyWayForward`) is not asserted. The generic `configengine` and `configreg` tests cover the mechanism, so this is not a finding.
8. `Config.DenyAgentTool`, `Spec.AllowAgentTool` and the template key are removed.
   - Pinned by `TestLoadConfig` (the `want` struct) and `TestModules_EveryTemplateKeyClassified` (the `claude` entry in `wantLocalDenied`).
   - The orch spec keeps `ForkSubagents`; `orchcli/start_test.go` pins it in two places.
9. Stencil prose: exempt.
10. The smoke tests.
    - `TestSmokeGuardrailDeniesHeadlessClaude` is the live proof of the deny and steer.
    - `TestSmokeBurlerClusterRogueFork` still pins the audit's Agent-call check.

**Findings**

- **F-T1 (LOW, consistency)**: the old key move `shuttle.claude_deny_agent_tool → claude.deny_agent_tool` is still declared, and no test pins how it behaves now.
  - Location: `internal/configreg/keymove.go:20`.
  - The `configsync_test.go` diff (about lines 2039–2182) swapped every `deny_agent_tool` expectation for `deny_ask_user_question`. That removed the only reconcile coverage of the old shuttle key arriving at the now-retired key.
  - `internal/shuttleengine/config_test.go:198,240` only exercises the shuttle side.
  - Evidence: the `configsync_test.go` diff, and the `keymove.go` read in the handler's context.
  - Fix: add a `configsync` row showing a shuttle `claude_deny_agent_tool` is dropped from the moved claude layer without error, or remove the move.
- **F-T2 (NIT, consistency)**: the "no Agent entry" assertion is duplicated.
  - Location: `internal/shuttleengine/claudeengine/settings_test.go`, `TestBuildSettings_DenyToggleMatrix` (`hasMatcher("Agent")`) and `TestBuildSettings_NoAgentHookAndForkGuard`.
  - The matrix assertion adds nothing once the golden file and the dedicated test pin it.
  - Fix: drop the `hasMatcher("Agent")` check from the matrix test, which now has no Agent-related column.

Gaps I found beyond the two findings: none.

**Tool calls**
1. `sed -n 84,125p config_test.go; grep -n "\.Forms"` in `internal/shellguard`: to see whether the local-override refusal is asserted and whether `Forms` rows are exercised by a mechanical test. The `sed` showed the refusal loop. The `grep` returned nothing in `shellguard`, so I did not confirm that `Forms` is checked anywhere.
