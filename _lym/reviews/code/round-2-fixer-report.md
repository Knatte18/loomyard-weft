# Round 2 fixer-report

## Changes

- F1 (`9530f686`): added a gone-waiter row and a live-waiter row to `TestAgentActivity_LiveLockWaitReadsAsActivity` and `TestLiveRunStrands_SkipsRunWithGoneWaiter` in `internal/loomcli/reviewstatus_test.go`, with the helper `writeRunningShuttleRun`.
  Fixed as proposed.
- F2 (`143425d7`): reworded the `judgeAgents` comment in `internal/battenshed/notice.go` and the `Activity` field and `AgentActivity` type comments in `internal/battenshed/deps.go`.
  Fixed as proposed.
- F2, self-check follow-up: reworded the nil-`Activity` comments in `internal/battenshed/deps.go` and `internal/battenshed/innerrun.go` the same way.

`lym gate test ./internal/loomcli ./internal/battenshed` passed (tier 0) after the F1 and first F2 commits.
The follow-up is a comment-only change and was not rerun.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, stale "no live run" in the nil-`Activity` comments (`deps.go`, `innerrun.go`); fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: LOW, the same stale comments; merged with the fix-consistency finding and fixed as proposed.

## Disputed

None.

## Deferred

None.
