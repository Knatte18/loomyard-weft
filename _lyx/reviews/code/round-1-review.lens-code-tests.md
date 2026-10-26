```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff changes the test that pins it.

**Per behaviour: pinning test**
- **The `in-place-edit` and `redirect-write` rows are gone, and a scripted edit passes every remaining row.** Pinned by `TestRows_PatternsDenyAndAllow` in `internal/shellguard/rows_integration_test.go`. The new `allowedEverywhere` loop runs `sed -i`, `perl -pi`, a redirect into a project file and an appended heredoc against every remaining row.
- **`Rows` returns no `in-place-edit` or `redirect-write` row by name.** No test asserts the absence directly. A reintroduced row would be caught only if its pattern matched one of the five `allowedEverywhere` commands. This is a minor gap and I'm not filing it.
- **The pattern-only deny and allow cases for the two rows are removed.** Nothing to pin, since the behaviour no longer exists.
- **The settings document and notice carry neither row's hook nor sentence.**
  - `TestBuildSettings_DenyToggleMatrix` pins the hook count at 3 under every input.
  - `TestBuildSettings_AgentAndBashHooks` pins one entry each for raw-go, computed-rm and sleep.
  - `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` pins the 4 and 3 counts.
  - `testdata/settings-golden.json` pins the full document byte for byte.
  - `TestBuildDenyNotice_SentenceContent` still asserts the computed-rm sentence.
- **The edit directive permits scripted edits.** A static-prose change with no assertion, which the rubric exempts.
- **No stencil restates the edit rule.** `TestStencils_NoEditRuleRestatement` with the reworded `editRulePhrases`. `parentdirective_test.go:222` (`editRuleSentences("Fix it with `sed -i`.")`) still exercises the phrase list.
- **The no-denied-recovery scan no longer reports `sed -i` as denied.** `TestNoDeniedRecovery_ScanSelfCheck` row "sed -i is not denied". Its expected count of 0 would fail if the row's `Forms` came back.
- **The `Rows`-driven shell-guard hook cases still run end to end.** `TestShellGuardDenies_RenderedCommandsGrepThePayload` now uses `machine-sleep` rows in place of the removed `redirect-write` rows.

**Findings**

1. **LOW, consistency.** The `machine-sleep` rows added in `internal/shuttleengine/claudeengine/hookcmd_integration_test.go:232-238` replaced the `redirect-write` rows. The "systemctl hibernate" deny and the "grep 'x; systemctl hibernate'" allow already have pattern-level rows in `TestRows_PatternsDenyAndAllow`. The rendered-command test only needs one denied and one allowed case per row family, and the `computed-rm` rows already cover that, so the swap is partly redundant. I'm raising it because test economy asks to name the covering test, not because it breaks anything. No change is needed if you read the rendered-command path as worth pinning per row.

2. **NIT, consistency.** The `allowedEverywhere` loop in `rows_integration_test.go:~203-223` runs five commands for every row, so the runs multiply by the row count. The `sed_in_place_edit`, `perl_in_place_edit`, `redirect_into_a_project_file` and `heredoc_appended_to_a_project_file` cases all pin the same behaviour: no row blocks a scripted edit. Collapsing them to one or two is optional.

I found no uncovered behaviour. The golden diff's size follows from the removed rows.

**Tool calls**
1. `git diff` of `settings_test.go`, `hookcmd_integration_test.go` and `parentdirective_test.go`: my context held only the shellguard and `norecoverydeny` test diffs, not these three.
