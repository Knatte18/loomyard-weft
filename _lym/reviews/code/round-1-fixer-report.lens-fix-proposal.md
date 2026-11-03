```yaml
lens: fix-proposal
outcome: reported
```

Findings (lens fix-proposal; F1–F7 were all fixed as proposed, none disputed):

1. **LOW**: `internal/loomcli/wiring_test.go`, new `wantCappedTimeout`. F5's proposal ("a keep-warm cap and a `review_timeout_min` that make the cap bind") is carried out by hard-coding `shuttleengine.Config{StartRetries: 1, StartupTimeoutS: 90}`. Those numbers are copied from shuttle's template, not read from the `shuttleCfg` that `wire` loads. If the template's `start_retries` or `startup_timeout_s` changes, the test fails with a confusing 2m45s-vs-4m55s mismatch and no pointer to the template. Evidence: `internal/shuttleengine/*.yaml` lines 5–6 hold `startup_timeout_s: 90` and `start_retries: 1`. The comment names the dependency, but nothing enforces it. Deriving the values from `shuttleengine.LoadConfig` over the template would remove the coupling.

2. **LOW**: F2's proposal gives the unstartable seat the probe directory's paths (`probeDirOf`). `Validate` accepts them only because it checks that the paths are non-blank. The `Values` of a skipped seat now name `round-probe/instruction-*.md`, files that are never written. That is harmless today, because `Run` never renders the seat's stencil for a skipped seat. The proposal creates a latent trap if rendering ever moves before the skip branch. The `unstartedAssist` doc comment states the reason but not that the paths must never be read.

No breakage found in the other proposals:
- **F1**: the exported `ReviewRolePrefix` and `burlerDirName` cover every literal that `grep` showed.
- **F3**: `want == 0` returning `room` is consistent with the "zero seat timeout reads as the chair's" premise. The gate passed with the new row.
- **F4**: the `stopWaitErr` fake yields the `stopped` ending the review described.
- **F6**: the deleted test's rows stay covered by `loomengine/config_test.go:609-611` and `tapestry_test.go:259-260`. The accept-path rows (nil list, empty entry, mixed list) are covered by the load rows at `config_test.go:220` and `tapestry_test.go:144`, plus `review_test.go:166`.
- **F7**: the docs wording matches the keys in the template. I did not check whether the assist keys are local-layer open or closed.

Tool calls:
1. `grep plan_assist_reviewer_model internal/loomengine/*_test.go`: to confirm that the tapestry and plan rows cover the deleted `TestValidateAssistModelList`; my context held the loom rows but not the tapestry test file.
