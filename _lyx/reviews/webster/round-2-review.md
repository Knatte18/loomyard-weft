---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/battenshed/teardown_test.go:306
    summary: TestWorktreeTeardown_FailureNoticeOncePerEpisode hand-rolls logger capture instead of using the logcapture kit that the package's other tests use
---

### [LOW] Hand-rolled logger capture duplicates the logcapture kit

**Location:** `internal/battenshed/teardown_test.go:306`
**Issue:** The new test builds a `bytes.Buffer`, calls `logger.SetOutput` and restores `os.Stderr` in `t.Cleanup`.
`internal/testkit/logcapture.Capture` does exactly this, and `notice_test.go` and `innerrun_publishretry_test.go` in the same package already use it.
`PATTERN-testkit` says shared test support is never duplicated per package.
The test stays serial because of the process-global logger, which its doc comment states; that part is correct.
The only cost of the hand-rolled form is the extra `bytes` and `logger` imports and a second place that knows how to restore the logger.
**Fix:** Replace the three setup lines with `logs := logcapture.Capture(t)`, read `logs.String()` in the assertion, and drop the `bytes` and `logger` imports.

## Focus judgments

- Publish-retry tests: every top-level test in `innerrun_publishretry_test.go` that builds a `retryHarness` calls `t.Parallel`, except `TestInnerRun_PublishRetryDeclinesRecordsThatDoNotMatchTheHalt`.
  That one calls `logcapture.Capture`, which replaces the process-global logger output, and its doc comment says so.
  No test in the file uses `t.Setenv` or `t.Chdir`.
  Not a defect.
- `land.go` stop decision: `stop` and `pause` pass an explicit `notify` flag to `stuck`, and the wait helpers return a `paused` bool beside the reason.
  Nothing inspects the wording of a reason string.
  Not a defect.

`lyx gate test ./internal/battenshed` passes.
