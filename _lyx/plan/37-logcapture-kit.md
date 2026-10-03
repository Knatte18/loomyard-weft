# Card 37 — logcapture-kit

**Create:**
- `testkit/logcapture/logcapture.go`
- `testkit/logcapture/logcapture_test.go`

**Uses:**
- `testkit/doc.go`

**Intent:**
Provide one shared logger capture (#324 §6) for the per-package copies card 38 converts.

`logcapture.go` imports only `internal/logger` and exports `Capture(t testing.TB) *Buffer`, which points `logger` output at a fresh buffer and restores `os.Stderr` (and the default verbosity) in `t.Cleanup`.
`Buffer` is safe for concurrent writes and reads (`String()`), because the watch-loop and wait tests log from background goroutines.
`CaptureVerbose(t)` additionally raises verbosity to Info for the duration, covering the copies that call `logger.SetVerbosity(1)`.
Its package doc (D11) says the kit redirects `internal/logger` output for one test and restores it afterwards, imports only `internal/logger`, and so cannot serve `logger`'s own tests or the packages `logger` depends on.

`logcapture_test.go` asserts captured text, restoration of the default output after the test, and that concurrent writes and `String()` calls pass under `go test -race`; the card's `Verify` runs it under the race detector, which the plain package run does not.

**Verify:** go test -race ./internal/testkit/logcapture/...
