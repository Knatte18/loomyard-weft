# Card 1 — Logger discards its stderr half under go test

**Create:**
- `internal/logger#DefaultOutput`
- `internal/logger#TestDefaultOutput_DiscardsUnderGoTest`

**Edit:**
- `internal/logger#out`
- `internal/testkit/logcapture#Capture`
- `cmd/lyx#redirectSink`
- `internal/battenshed#TestInnerRun_DoneFinishesPastGraceWithLiveDriverAndWarns`
- `internal/battenshed#TestInnerRun_HaltWarnsOncePerEpisode`
- `internal/battenshed#TestInnerRun_ReHaltAfterObservedResumeWarnsAgainAtTheSameHistoryLength`
- `internal/battenshed#TestInnerRun_HaltedRevivesADeadDriverOncePerEpisode`
- `internal/battenshed#TestInnerRun_StrandReadErrorSkipsTheReviveWithAWarn`
- `internal/battenshed#TestInnerRun_TerminalOpenErrorIsWarnedNotEscalated`
- `internal/battenshed#TestInnerRun_RunningArmReviewWaitNote`
- `internal/battenshed#TestInnerRun_WaitEndsOnlyOnAnEvent`
- `internal/battenshed#TestInnerRun_RecordsWhetherItHoldsTheWatchedMarker`
- `internal/battenshed#TestNotice_DeliveryIsRecordedOnlyWhenQueuedAndRetriedWithinItsCaps`
- `internal/burlerengine#TestComposePrompt_FocusDirective`
- `internal/burlerengine#TestWarnIfFocusMarkerAbsent`
- `internal/configreg_test#loadFilled`
- `internal/fabricengine_test#TestAddRollback_RefusedWarpBranchDeletionLogsWarn`
- `internal/parentreview#TestGate_LivenessErrorHoldsAndWarnsOncePerError`
- `internal/parentreview#TestRecordDelivered`
- `internal/reedengine#TestTmuxConfBootGuard`
- `internal/shuttleengine/claudeengine#TestSessionNameDrift_MissingTranscriptReportsNoneAndLogs`
- `internal/shuttleengine/claudeengine#TestSessionNameDrift_UnreadableTranscriptReportsNoneAndLogs`
- `internal/stencilcli#stepDriftWarningNeverBlocksExitCode`
- `internal/stencilstore#TestWarnPortBackDrift_EmitsOneLineListingTheDriftedStencils`
- `internal/treadleengine#TestEngine_HandoffLifecycle_InvalidHandoffFallback`
- `internal/treadleengine#TestEngine_JudgeSkippedRoundReadsNoHandoffFiles`
- `internal/treadleengine#TestEngine_RecordedHandoffCorruptedMidBlockWarnsUnderCallerName`
- `internal/verifyrun#runCapture`
- `internal/websterengine_test#captureLogs`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D13: under `go test` the logger's stderr half defaults to `io.Discard`, so a package test run whose code logs at Warn leaves no logger output on stderr.

`DefaultOutput` returns `io.Discard` when `testing.Testing()` is true and `os.Stderr` otherwise, the same test-time switch `resolveSinkDirLocked` already applies to the durable half.
`out` is initialised from `DefaultOutput()`, so `currentStderr` is built over it at package init.
The durable sink is untouched, `SetOutput` keeps its contract, and a test that asserts on log output still calls `SetOutput` (or `logcapture.Capture`) itself.
`configureFromEnv`'s `LYX_LOG_FILE` fallback leaves the stderr half at its default, which under `go test` is now the discard writer.
The doc text in `logger.go` that says the stderr sink defaults to `os.Stderr` (the file header comment, the package doc's opening paragraph and `out`'s own comment) says it defaults to `DefaultOutput`, and the package doc states the trade-off D13 names: a developer running a failing test with `-v` sees no logger lines unless that test redirects the logger.

Every test that restores the stderr half after a capture restores the default instead of `os.Stderr`, or the restore would re-open stderr for every later test in the same binary:
`logcapture.Capture`'s cleanup calls `logger.SetOutput(logger.DefaultOutput())`, and each listed test function or helper replaces its `logger.SetOutput(os.Stderr)` cleanup with `logger.SetOutput(logger.DefaultOutput())`.
A listed file whose only use of `os` was that call drops the import.
`tools/sandbox/main.go`'s `SetOutput(os.Stderr)` is production code outside a test binary and stays.

`TestDefaultOutput_DiscardsUnderGoTest`, in `internal/logger/logger_test.go`, pins the switch: `DefaultOutput()` is `io.Discard` inside the test binary, and so is `originalOut`, the value `out` held at package init, so a default or an init that still chose `os.Stderr` fails it.
It reads no shared state it mutates, so it calls `t.Parallel`.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/logger` row rises from 35 to 36.

**ImpactSummary:** one unexported var and one new exported function in `internal/logger`; every other target is a test cleanup or the shared capture kit, so no production behavior outside a test binary changes.

**Commit:** 1: logger: discard the stderr half under go test
