# Card 38 — logcapture-callers

**Edit:**
- `configengine/config_test.go`
- `ideengine/spawn_clean_integration_test.go`
- `landingshed/publish_test.go`
- `landingshed/publish_verify_test.go`
- `landingshed/verifygate_test.go`
- `loomshed/gatefindings_test.go`
- `mergeresolve/mergeresolve_test.go`
- `reedengine/logcapture_test.go`
- `reedengine/attach_test.go`
- `reedengine/ensuresession_integration_test.go`
- `reedengine/launchscript_cleanup_test.go`
- `reedengine/naming_integration_test.go`
- `reedengine/namerepair_test.go`
- `reedengine/panebin_test.go`
- `reedengine/reconcile_test.go`
- `reedengine/spawn_test.go`
- `reedengine/watchloop_test.go`
- `selfreportengine/selfreport_test.go`
- `shedadapters/bouncer_judge_test.go`
- `shedadapters/burler_test.go`
- `shedadapters/singlellm_test.go`
- `shuttleengine/wait_test.go`
- `websterengine/runverify_test.go`

**Uses:**
- `testkit/logcapture/logcapture.go`

**Intent:**
Replace every package's own logger-capture helper with card 37's `logcapture` kit (#324 §6).

Each defining file's helper (`captureLogOutput`, `runverifyCapture` and the other local `logger.SetOutput` + `t.Cleanup` wrappers) is deleted, and every caller, in the defining file or a sibling file listed here, uses the kit.
A test in these files that redirects the logger inline with the identical shape converts too; an inline capture with a different shape (another writer, no restore) stays.
`internal/logger`'s own tests and the packages `logger` depends on cannot import the kit and are left alone.
Apply D10 to every touched test.

**ImpactSummary:** Test files across many packages swap a local log-capture helper for the kit; captured content and assertions are unchanged.
