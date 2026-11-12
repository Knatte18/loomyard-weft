```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour**
1. `--advisor` flows into `Spec.Advisor`, and an absent flag leaves it empty. Pinned by `TestRunCmd_ModelSpec`, row `ClaudeAliasWithAdvisor` (`internal/shuttlecli/cli_test.go`). The new `LastSpec.Advisor` assertion also runs on every other row, so those rows pin the empty case. The `Spec.Advisor` field itself is covered by the same test.
2. claudeengine writes `advisorModel` into the run's `settings.json` when `Spec.Advisor` is set, and omits the key when it is empty. Pinned by `TestPrepare_AdvisorReachesSettings`, rows `NoAdvisor` and `Advisor` (`settings_test.go`). It goes through `Prepare`, so it also covers the `claudeengine.go:224` wiring.
3. A launch without an advisor stays byte-identical to today's. Pinned by `TestBuildSettings_MatchesGolden`, which passes `""` and compares against the unchanged golden. `TestPrepare_AdvisorReachesSettings/NoAdvisor` also pins the key's absence.
4. codexengine refuses a spec with an advisor, before writing anything, and names the way forward. Pinned by `TestPrepare_RefusesBeforeWriting`, row `Advisor` (`prepare_test.go`). It asserts the `advisor "opus"` and `way forward` substrings. It uses the existing table's before-writing check, which I did not re-read.
5. `buildSettings` takes one more argument, `advisorModel`. All 12 or so call sites were updated mechanically, and the build gate decides whether they compile. No separate behaviour.
6. The resume line keeps the advisor because it names the same `settings.json`. This is a claim in `doc.go` and the summary, not new code. The existing launch/resume consistency test (`TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`) pins the shared settings path. No new row is needed.
7. The help text, `docs/overview.md`, and the `doc.go` text are static prose, so they are exempt.

**Findings:** none.

**Test economy:** `TestPrepare_AdvisorReachesSettings` is reported unique, and the other two additions are rows in existing tables. I found no redundant test or per-helper test.

**Tool calls:** none. Everything I needed was already in context.
