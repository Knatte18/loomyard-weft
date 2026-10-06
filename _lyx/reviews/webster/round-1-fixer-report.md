---
fixed: [F1, F2, F3, F4, F5]
deferred: []
---

All five findings of `round-1-review.md` are fixed, each in its own commit on the task branch, none pushed.

## Changes

- **F1** (`ea0ca3a33`): broke every flagged comment at its independent-clause boundary in `run.go`, `claudeengine/skillload.go`, `shuttleengine/doc.go`, `orchengine/doc.go`, `orchengine/watcher.go` and `orchengine/watcher_test.go`.
  Re-flowed the `integration_driverbootstrap_test.go` stub comment so each sentence sits on one line.
  Untouched lines are as they were.
- **F2** (`17a7f9f40`): added `t.Parallel()` to `TestWatcher_ReloadRestartAroundTheRetryStep` and `TestWatcher_ReloadReadsAnOldStyleStepAsThePointer`, after `go test -race -count=3` passed on the reload tests.
- **F3** (`840510cf7`): turned the old-style step test into a table, renamed `TestWatcher_ReloadReadsAnUnreadableStepAsThePointer`, with a row for a persisted retry step with an empty `reload_retry`.
  Dropping the `len(st.ReloadRetry) > 0` condition in `reloadStep` makes the new row fail, which was checked before the condition was restored.
- **F4** (`9a4ae4305`): extended the rows of `TestWatcher_SkillSkipCauses` with the expected `skill skipped` count and log fragments (`skill`, `cause`, `skill load unverified`), asserted through `logcapture` as `TestStartup_SkillLoadTurns` does.
  The test no longer calls `t.Parallel`, with a comment naming the process-global logger.
  Changing the `not loaded` cause to another string makes the test fail, which was checked before the string was restored.
- **F5** (`1341b4b5f`): `ResetForFreshLaunch` and `Watcher.toIdle` assign `ReloadStepSkills`, not the literal `0`.

## Verification

`go build ./...`, the plan's `go test` under no tag and `integration`, and `go vet` under `llm` and `tmux` pass for the plan's verify packages; `grep -rn 'SkillUnknown\|SkillLoadSequence' --include=*.go internal` is empty.
The `tmux`-tag tests passed in job A, and this round changed only comments and orch test code since.

## Deferred

Nothing deferred.
