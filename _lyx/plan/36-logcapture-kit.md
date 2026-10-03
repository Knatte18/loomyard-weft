# Card 36 — logcapture-kit

**Create:**
- `testkit/logcapture/logcapture.go`
- `testkit/logcapture/logcapture_test.go`

**Edit:**
- `configengine/config_test.go`
- `ideengine/spawn_clean_integration_test.go`
- `landingshed/publish_test.go`
- `loomshed/gatefindings_test.go`
- `mergeresolve/mergeresolve_test.go`
- `reedengine/logcapture_test.go`
- `reedengine/watchloop_test.go`
- `selfreportengine/selfreport_test.go`
- `shedadapters/bouncer_judge_test.go`
- `shedadapters/singlellm_test.go`
- `shuttleengine/wait_test.go`
- `websterengine/runverify_test.go`

**Uses:**
- `testkit/doc.go`

**Intent:**
Replace every package's own logger-capture helper with one kit (#324 §6).

`logcapture.go` imports only `internal/logger` and exports `Capture(t testing.TB) *Buffer`, which points `logger` output at a fresh buffer and restores `os.Stderr` (and the default verbosity) in `t.Cleanup`.
`Buffer` is safe for concurrent writes and reads (`String()`), because the watch-loop and wait tests log from background goroutines.
`CaptureVerbose(t)` additionally raises verbosity to Info for the duration, covering the copies that call `logger.SetVerbosity(1)`.

Each listed file's helper (`captureLogOutput`, `runverifyCapture` and the other local `logger.SetOutput` + `t.Cleanup` wrappers) is deleted and its callers use the kit.
A test in these files that redirects the logger inline with the identical shape converts too; an inline capture with a different shape (another writer, no restore) stays.
`internal/logger`'s own tests and the packages `logger` depends on cannot import the kit and are left alone.

`logcapture_test.go` asserts captured text, restoration of the default output after the test, and that concurrent writes and `String()` calls pass under `go test -race`.
Apply D10 to every touched test.

**ImpactSummary:** Test files across many packages swap a local log-capture helper for the kit; captured content and assertions are unchanged.
