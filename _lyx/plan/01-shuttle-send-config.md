# Card 1 — Send window config keys

**Edit:**
- `internal/shuttleengine#Config`
- `internal/shuttleengine#LoadConfig`
- `internal/shuttleengine/template.yaml`
- `internal/shuttleengine/config_test.go`

**Intent:**
Add the two send windows of D9 to shuttle's config.
`Config` gains `SendReadyTimeoutS int` (`yaml:"send_ready_timeout_s"`, the idle wait of D1) and `SubmitConfirmTimeoutS int` (`yaml:"submit_confirm_timeout_s"`, the submit window of D3), each with a field comment in its neighbours' style.
`LoadConfig` refuses 0 or a negative value for either key with an error naming that key, as it does for `background_shell_wait_min`.

`template.yaml` gains `send_ready_timeout_s: 60` and `submit_confirm_timeout_s: 30`, with comments in the neighbours' style that say what each window bounds and that 0 or negative is a config error.
The comments of `submit_settle_ms` and `submit_redraw_settle_ms` are rewritten to their new meaning: the interval between the two input-box reads that must agree before Enter (D2), and the first step of the doubling confirm interval after an Enter (D3); both keys keep their names, defaults and validation.

`config_test.go` extends `TestLoadConfig_TemplateDefaultsResolve` with the two template defaults and extends the table of `TestLoadConfig_SubmitSettleKeys` (or the test that already tables per-key refusals) with rows for 0 and a negative value of each new key alone, asserting the error names that key.

**ImpactSummary:** Two new fields and two new refusals on shuttle's config; every existing key keeps its value, and no reader of the new fields exists yet.

**Commit:** 1: shuttle: add the send-ready and submit-confirm windows to config
