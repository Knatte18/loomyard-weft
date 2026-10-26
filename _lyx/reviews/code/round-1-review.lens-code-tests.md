```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (code-tests lens)**

- **`in-place-edit` and `redirect-write` rows gone from `Rows`:** pinned by `TestRows_PatternsDenyAndAllow` (`internal/shellguard/rows_integration_test.go`). Its per-row allow loop now runs `sed -i`, `perl -pi`, a redirect and a heredoc append against every remaining row. The lookup `shellRow(t, "in-place-edit")` in `settings_test.go` can no longer resolve, so a re-added row would break those tests only if they referenced it. The settings golden and the `PreToolUse` counts catch a re-added row (see the next bullet).
- **Settings document and notice carry neither row's hook or sentence:**
  - `TestBuildSettings_DenyToggleMatrix` pins the entry count at 3 (`settings_test.go` ~L319).
  - `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` pins the counts at 4 and 3.
  - `TestBuildSettings_AgentAndBashHooks` still demands exactly one each of raw-go, computed-rm and sleep.
  - The settings golden is pinned byte for byte.
  - A sentence from a removed row appearing in the notice is not asserted absent by name. It would still surface through the golden, if the golden covers the notice (not checked).
- **Remaining rows unchanged (python, raw-go, refuse-raw, computed-rm, machine-sleep):** their existing table rows in `TestRows_PatternsDenyAndAllow` are untouched, and the `payloadDenied` end-of-payload rows for those five remain.
- **`hookcmd_integration_test.go` `TestShellGuardDenies_RenderedCommandsGrepThePayload`:** the removed `redirect-write` rows were replaced with three `machine-sleep` rows. I did not open the existing machine-sleep rows in `TestRows_PatternsDenyAndAllow` to compare. If they already hold the same deny, allow and grep-quote cases, the replacement is only the rendered-hook-grep boundary check, which is the test's own job.
- **Edit directive permits scripted edits:** pinned by `TestDirective_RendersTheRuleWithoutBannerOrMarkers`, `TestStencils_OpeningStencilsRenderEditDirective` and `TestStencils_NoEditRuleRestatement`. Dropping `replace_all` from `editRulePhrases` is consistent with the new text.
- **No-denied-recovery scan no longer reports `sed -i`:** pinned by the `"sed -i is not denied"` row (`cmd/lyx/norecoverydeny_test.go` ~L228).

**Findings**

1. **LOW / consistency** — `internal/editdirective/editdirective_test.go` L111–118.
   - The diff adds assertions on the directive's static prose: `"by script with \`sed -i\`"` and the absence of `"never \`sed"`.
   - The rubric's test economy forbids a new assertion on a stencil's static prose, and this one pins wording, not behaviour.
   - Fix: revert to the single `"Edit or Write"` anchor, with the `{{` marker check unchanged. `TestStencils_NoEditRuleRestatement` and `editRulePhrases` in `contracts/stencils` already cover the single-statement rule.

2. **NIT / consistency** — `internal/shellguard/rows_integration_test.go`, `allowedEverywhere` (~L206–211).
   - The rows `redirect_into_a_project_file` (`echo x > docs/a.md`) and `heredoc_appended_to_a_project_file` (`cat >> docs/x.md <<EOF…`) are both "a redirect into a project file passes every row".
   - A redirect row is redundant with a heredoc row on that point: the removed `redirect-write` row's target check had no heredoc-specific branch left to pin.
   - Fix: drop one of the two.

No other behaviour the diff adds lacks a pinning test.

**Tool calls:** 1. `git diff 6865efba..HEAD` of the shellguard and claudeengine settings test files. The manifest held only the persisted-output preview for those two files.
