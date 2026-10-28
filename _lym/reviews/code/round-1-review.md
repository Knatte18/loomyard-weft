---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/loomcli/looparm.go:168
    origin: lens:code-affected-set
    summary: A running record with a dead waiter now counts as a live agent run in the loop-arm activity check and in the decision notice recipients, beyond D3's bound of the idle notice only.
    fix: Add a WaiterGone bool to AgentActivity in internal/shuttleengine/activity.go, set where ReadAgentActivity zeroes the API-error and usage-limit parts (activity.go:113). Make agentActivityFor (looparm.go:168-181) and liveRunStrands (internal/loomcli/decision.go:855) skip a reading with WaiterGone, so only battencli/wire.go reads a waiterless run. Reword the "live" doc comments of liveRunStrands (decision.go:844), agentActivityFor (looparm.go:125) and internal/battenshed/doc.go (lines 20-28, "live run" and the "no agent activity readable" fallback) to say a running run is read and a waiterless one carries activity only. Add a ReadAgentActivity row that sets WaiterGone for the waiterless case.
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/carryover_test.go:52
    origin: lens:code-tests
    summary: The carry-over table pins neither the died and timeout predecessors nor the choice of the newest run when a role key has several runs.
    fix: In TestStart_CarriesOverOperatorInstructions add rows for a predecessor with Outcome OutcomeDied and one with OutcomeTimeout, each carrying its operator instruction. Add a row seeding two runs of one role key, an older unfinished one beside a newer done one, which must carry nothing. This needs the table to seed more than the single run-old directory.
  - id: F3
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/signals.go:197
    origin: lens:code-correctness
    summary: typedInstruction treats any prompt starting with a slash as a slash command, so an operator instruction that opens with an absolute path is not carried over.
    fix: In typedInstruction treat a prompt as a command only when its first whitespace-delimited token starts with "/" and holds no further "/". Add a TestParseSessionSignals row for a prompt such as "/tmp/x.go fails to build" that yields it as the instruction.
  - id: F4
    severity: LOW
    class: design
    location: internal/shuttleengine/carryover.go:70
    origin: lens:code-correctness
    summary: operatorInstructions always skips the first turn start past the prompt offset as the run's prompt, so when the prompt was never delivered the first operator instruction is dropped instead.
    fix: In operatorInstructions skip the first turn start only when its instruction is the run's prompt, or empty. Use the prompt line the run's pending-prompt record holds, or skip it only when the run's events show the prompt delivered. If neither is readable cheaply, say in the function's doc comment that a run that died before its prompt landed loses its first instruction.
  - id: F5
    severity: LOW
    class: design
    location: internal/shuttleengine/carryover.go:17
    origin: lens:code-tests
    summary: The failed sent-text append not blocking a send, the Warn log of the carry-over, and the carry-over kept across StartRetries attempts with one notice have no test row.
    fix: Add a TestRun_Send_RecordsTheSentText row where the run directory cannot be written, and assert Send succeeds. In TestStart_CarriesOverOperatorInstructions assert the Warn log line through the test logger, and add a start-retry row asserting the carried block stays in the prompt of the second attempt and one notice goes out after success.
---

### [MEDIUM] Dead-waiter runs count as live outside the idle notice

**Location:** internal/loomcli/looparm.go:168, internal/loomcli/decision.go:855, internal/battenshed/doc.go:20-28
**Issue:** D3 bounds the effect of reading waiterless runs to the idle notice naming an agent instead of "no agent activity readable".
`ReadAgentActivity` has three callers.
The two besides batten read presence as liveness: `agentActivityFor` returns `len(readings) > 0` as "an agent is live", and `liveRunStrands` hands every reading's strand to the decision notice poster.
A record stuck at running with a dead waiter and a gone agent now keeps both true until the record changes, where it was skipped before.
Their doc comments and `battenshed/doc.go` still say "live run", which the change makes wrong.

### [MEDIUM] Carry-over selection is not fully pinned by tests

**Location:** internal/shuttleengine/carryover_test.go:52
**Issue:** `unfinished` accepts stopped, died, timeout and waiterless-running, but only stopped and waiterless-running have rows.
The table seeds one predecessor, so the rule that only the newest run of the key counts, and a done run hiding an older unfinished one, is not exercised.
Breaking either would pass every test in the diff.

### [LOW] An instruction opening with a path reads as a slash command

**Location:** internal/shuttleengine/claudeengine/signals.go:197
**Issue:** `typedInstruction` drops any trimmed prompt starting with "/".
An operator typing "/tmp/x.go fails to build, fix it" loses that instruction on a re-spawn.
Only `/compact` is pinned by a test.

### [LOW] A run that died before its prompt landed loses its first instruction

**Location:** internal/shuttleengine/carryover.go:70
**Issue:** The first turn start past the prompt offset is skipped unconditionally as the run's prompt.
For a deferred-prompt run that died before the prompt was typed, that first turn start is an operator instruction, and it is dropped.
The loss is bounded to one instruction.

### [LOW] Failure and retry paths of the carry-over and sent-text record are untested

**Location:** internal/shuttleengine/carryover.go:17
**Issue:** D4's "a failed append does not block the send" and the Warn log the focus asks about have no row.
Neither has the carried block surviving a start retry with one notice.

## Rejected

- Lens code-correctness F4, same-second `CreatedAt` tie-break by directory name: it needs two unfinished runs of one role created in the same second, which only start-retry attempts do, and those carry the same instructions, so no wrong text is carried.
- Lens code-affected-set F3, `SetNotifier` doc paragraph: the paragraph at `doc.go:89` only says a nil notifier holds silently and enumerates no uses.
- Lens code-tests F6, no row decoding an older `run.json`: both new fields are `omitempty` and optional, and the existing record rows read records without them.
- Lens code-affected-set F1 and F2 and code-correctness F2 are kept merged as F1, since they describe the same defect.
- D7 and D8: the settings carry no subagent hook and no auto-continue key, and the change description and `claudeengine/doc.go` name both as left out with their reasons.
