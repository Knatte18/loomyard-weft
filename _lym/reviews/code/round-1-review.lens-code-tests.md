```yaml
lens: code-tests
outcome: reported
```

**Per behaviour, the test that pins it**

1. **Run document carries `"attribution":{"commit":"","pr":""}`, standalone (no creel flag dir), all interactive, fork and deny combinations.**
   - Pinned by `TestBuildSettings_MatchesGolden`, `settings_test.go:998`. It compares the document byte for byte against `settings-golden.json`, and the golden now carries the key in every entry.
   - It would fail if the key were dropped or changed. Every call passes `""` as the flag dir (`settings_test.go:1028`).
   - `TestSettings_CreditNoSession/RunStandalone` (`settings_test.go:83`) covers the same behaviour.

2. **Run document carries the attribution when a creel flag dir is set (hub run).**
   - Pinned only by `TestSettings_CreditNoSession/RunInAHub` (`settings_test.go:86`).
   - The golden never passes a flag dir, so no other test covers this path.
   - The path adds nothing to the attribution. `Attribution` is a struct field that `buildSettings` never touches, and the hub branch only changes the hooks (`settings.go:449-456`).

3. **Opened session's creel-only document carries the attribution.**
   - Pinned by `checkOpenCreelFiles`, `command_test.go:580`. It runs from `TestOpenLaunch`, a tier0 parallel test (`command_test.go:353`, call at `:511`), against the file `OpenLaunch` really writes.
   - `TestSettings_CreditNoSession/Opened` (`settings_test.go:89`) covers the same behaviour.

4. **Opened document holds only `hooks` and `attribution`, with hooks holding only Stop and PostToolUse.**
   - Pinned by `checkOpenCreelFiles`, `command_test.go:577-579`, with `len(keys) != 2`.
   - The hooks assertions that follow (`command_test.go:583-597`) are unchanged in content. The struct reshape only moves them under `doc.Hooks`, so none is weakened.

5. **The attribution key does not displace `promptSuggestionEnabled` or hooks in the run document.**
   - Pinned by `TestBuildSettings_PromptSuggestionOff` and the golden.

**Findings**

- **LOW, `consistency`, `settings_test.go:75-105`, `TestSettings_CreditNoSession`.**
  - Test economy: the test duplicates existing coverage.
  - `RunStandalone` repeats `TestBuildSettings_MatchesGolden`, which pins the whole run document including `attribution` in all 16 combinations (`settings_test.go:998`, `testdata/settings-golden.json`).
  - `Opened` repeats `checkOpenCreelFiles`, reached through `TestOpenLaunch` (`command_test.go:580-581`).
  - `RunInAHub` is the only row not covered elsewhere. It guards a path that cannot change the attribution, so it adds no behaviour of its own.
  - The summary's claim that the opened document "had no test of its own content" is wrong. `checkOpenCreelFiles` already asserted its hooks content before this diff.
  - Fix: delete the test. If the hub row is wanted, fold it in as one assertion in the existing golden or open-launch test.

- No behaviour is left uncovered, and no existing assertion is weakened (`command_test.go:558-597`, golden file).

**Tool calls**
1. Read `settings_test.go:990-1050`: the golden test's call arguments (flag dir `""`, all 16 combinations) were not in my context.
2. Grep of `command_test.go` for `TestOpenLaunch` and `checkOpenCreelFiles`: confirmed that `TestOpenLaunch` is tier0 and parallel, and that it calls the check.
