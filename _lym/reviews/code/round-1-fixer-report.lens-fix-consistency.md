```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F2 and F3 change set (`_lym/landing/summary.md`, `internal/configreg/configreg_test.go`) for agreement with its surroundings.

1. **LOW** — `_lym/landing/summary.md`, "## Tests" section (lines 36-39).
   - Summary: the section lists `TestLoadConfig_RemoteControl` and `TestFlagRemoteControlDefault` but not the new `TestModules_OrchRegistersRemoteControlReinterpretation`, which the F3 fix added.
   - Evidence: the new test is committed in 7cfd66753, and the "## Tests" section still stops at two bullets.
   - A reader of the summary would not learn that the orch row's registration is pinned.
   - A one-bullet addition would close this.

The F2 sentence in the summary (lines 19-20) agrees with the review's evidence: `ReconcileHubWideAt` seeds the absent file before `applyReinterpretations` reads it. The ID literal in the new test matches the ID the summary names, `orch-remote-control-off-by-default`. The orch row at `configreg.go:161` still carries exactly one `Reinterpretations` entry, as the test asserts. The gate passed on the new test.

No further tool calls made; my context already held every file I checked.
