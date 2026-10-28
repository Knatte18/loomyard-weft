---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: internal/loomcli/decision.go:862
    summary: The WaiterGone skip added to liveRunStrands and agentActivityFor has no test
    fix: Add a waiterless-run row to the table in internal/loomcli/reviewstatus_test.go TestAgentActivity_LiveLockWaitReadsAsActivity, with a run.json under the anchor's .lym/shuttle whose outcome is running and whose pid is dead (1<<30), expecting live false; add a case there with a live-pid run beside it expecting live true. Add one test of liveRunStrands with the same two run records, expecting only the live run's strand name.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/battenshed/notice.go:434
    summary: Doc comments in battenshed still say a live run where a waiterless running run is now read
    fix: In internal/battenshed/notice.go lines 434-435 reword "several live runs" to "several runs read" and "With no live run found" to "With no run read"; in internal/battenshed/deps.go reword the Activity field's "live agent runs" (line 223) to "running agent runs, those whose waiting process is gone included", and the AgentActivity type's "one live agent run" (line 314) to "one running agent run".
---

### [MEDIUM] The WaiterGone skip in loomcli has no test

**Location:** internal/loomcli/decision.go:862, internal/loomcli/looparm.go:176
**Issue:** The diff makes `ReadAgentActivity` return waiterless running runs, and adds a `WaiterGone` skip in both `liveRunStrands` and `agentActivityFor` so a decision notice or the review-status activity probe never counts a run no process waits on. That skip is the only guard keeping those two callers on their old behavior. No test in `internal/loomcli` reaches it: `grep WaiterGone` in the package's tests finds nothing, `liveRunStrands` has no test, and the `agentActivityFor` cases in `reviewstatus_test.go` write no shuttle run record. Dropping either `if` leaves every loomcli test green while a dead run's strand receives a decision notice or reads as live agent activity. The shuttleengine side is covered (`activity_test.go` pins `WaiterGone: true` readings); the callers' reaction to it is not.

### [LOW] Stale "live run" doc comments in battenshed

**Location:** internal/battenshed/notice.go:434, internal/battenshed/deps.go:223
**Issue:** The change reads waiterless running runs into the batten idle notice, and `battenshed/doc.go` and `battencli/wire.go` were reworded to match. Three comments of the same seam still describe the old rule. `judgeAgents` says "With several live runs" and "With no live run found, quiet falls back", but a waiterless run is now read and prevents the fall-back; the `Activity` field says it "reads what the task worktree's live agent runs have been doing", and the `AgentActivity` type says "one live agent run". The reading's own doc says the opposite: such a run is read but is not live.

## Focus answers

- **Waiterless run counted live elsewhere.** No. The batten idle notice reads the run, with its API-error and usage-limit parts zeroed, so no revive or usage-limit notice goes to it. The other `ReadAgentActivity` callers, `liveRunStrands` and `agentActivityFor`, skip `WaiterGone`. `ReadSessionStates` and `ReadAgentWaits` still gate on `runIsLive`. The new doc comments in `activity.go`, `looparm.go`, `decision.go`, `battenshed/doc.go` and `battencli/wire.go` match this; the stale ones in battenshed are F2.
- **Carry-over selection table.** The table pins stopped, died, timeout and waiterless running as carried, and done and live-waiter running as not carried. It pins that only the newest run of the role key counts, in both orders, with a done newer run hiding an unfinished older one. A tie on `CreatedAt` falling back to the directory name is unpinned, which is not worth a finding.
- **`typedInstruction`.** A prompt opening with an absolute multi-segment path is kept as an instruction, and `/compact` and `<task-notification>` are dropped, each with a row in `signals_test.go`.
- **First instruction of a run whose prompt never landed.** Kept: `operatorInstructions` matches the first turn start by the recorded prompt line when one exists, and the doc comment states it. The "died before its deferred prompt landed" and "delivered deferred prompt" rows pin both sides.
- **Send path and start path coverage.** A failed sent-text append not failing the send, with one Warn, is pinned. The carried block surviving a start retry with one notice and one carry-over Warn is pinned by `TestStart_CarryOverSurvivesAStartRetry`.
