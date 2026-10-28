# Fixer-report — code review round 1

## Changes

- **F1** fixed as proposed (commit `17fb7ae3`).
  `AgentActivity.WaiterGone` is set where `ReadAgentActivity` zeroes the API-error and usage-limit parts.
  `agentActivityFor` and `liveRunStrands` skip a `WaiterGone` reading, so only the battencli idle-notice reader still reads a waiterless run.
  The "live" wording of `liveRunStrands`, `agentActivityFor`, `battenshed/doc.go` and `shuttleengine/doc.go` is reworded, and the three `ReadAgentActivity` rows that read a waiterless run expect `WaiterGone: true`.
- **F2** fixed as proposed (commit `598e7992`).
  `TestStart_CarriesOverOperatorInstructions` seeds any number of predecessors through `seedPredecessor`, with rows for died, timeout, a done run newer than an unfinished one and the reverse.
- **F3** fixed as proposed (commit `f7cec7ac`).
  `typedInstruction` treats a prompt as a command only when its first word starts with `/` and holds no further `/`; `TestParseSessionSignals` has a `/tmp/x.go fails to build` row.
- **F4** fixed with the second option the `fix` key names, in commit `598e7992`.
  `operatorInstructions` skips as the run's prompt the first turn start when the record has no `PromptLine`, else the one whose instruction is that line.
  Rows cover a run that died before its deferred prompt landed and one whose deferred prompt was delivered.
- **F5** fixed as proposed (commit `598e7992`, then the self-check commit).
  `TestStart_CarryOverSurvivesAStartRetry` asserts the carried block in both launches' prompts, one notice after success and one Warn line.
  `TestRun_Send_SucceedsWhenTheSentTextCannotBeRecorded` asserts the send succeeds with an unwritable run directory and logs the failure once.
- Self-check commit: reworded the stale "live" comments (`AgentActivity`, `activity.go` header, `battencli/wire.go` `readAgentActivity`, `battenshed/doc.go` limits line and usage-limit episode line), widened the `WaiterGone` doc to the pid-less record, and noted the one-segment-path limit in `typedInstruction`.

## Self-check

- fix-consistency, MEDIUM, `battenshed/doc.go:37` "a run whose pid is dead is not read": fixed as proposed.
- fix-consistency, LOW, `AgentActivity` still "one live run" (with `activity.go` header, `wire.go:590`): fixed.
- fix-consistency, LOW, `battenshed/doc.go:26` "any live run": fixed.
- fix-affected-set, LOW, `AgentActivity` type comment: merged with the fix-consistency finding, fixed.
- fix-affected-set, LOW, `battencli/wire.go:590` comment: fixed.
- fix-proposal, LOW, `WaiterGone` doc claims more than a pid-less record shows: fixed, doc widened.
- fix-proposal, LOW, `/etc`-style one-segment path still read as a command: disputed, see below; the limit is noted in the doc comment.
- fix-code-correctness, LOW, send-failure row passes regardless of tolerance: fixed, moved to a serial test asserting the Warn line.
- fix-code-correctness, LOW, no test row for the `WaiterGone` skip in `agentActivityFor` and `liveRunStrands`: deferred, see below.
- All four forks returned a report.

## Disputed

- Fork finding (fix-proposal, LOW): a prompt such as "/etc is unreadable" is still dropped.
  Ground: every fix worse.
  Evidence: `/etc` and `/compact` are textually identical in shape, a single `/`-led word, and the hook payload carries nothing else to tell them apart, so any rule that keeps `/etc` also keeps `/compact` as an instruction.

## Deferred

- Fork finding (fix-code-correctness, LOW): a test row for the `WaiterGone` skip in `agentActivityFor` and `liveRunStrands`.
  Reason: both functions build their readings from the worktree's config directories and a real `shuttleengines` set with no seam to inject readings, so a row needs a full worktree and config fixture in `internal/loomcli`, beyond this round's change; the review asked only for the `ReadAgentActivity` row, which exists.
