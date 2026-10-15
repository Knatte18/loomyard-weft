# Card 5 — history-caller-field

**Create:**
- `internal/shedengine#CallerShed`
- `internal/shedengine#RecordResumer`
- `internal/shedengine#TestRecordResumer`

**Edit:**
- `internal/shedengine#HistoryEntry`
- `internal/shedengine#Status`
- `internal/shedengine#Shed`
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#GotoRequest`
- `internal/shedengine#Goto`
- `internal/shedengine#Relaunch`
- `internal/shedengine#Shed.persistTransient`
- `internal/shedengine#TestGoto_MovesBlockedRunToTarget`
- `internal/shedengine#TestRelaunch`
- `internal/shedengine#TestStep_ResumeFromHaltedStates`
- `internal/shedengine#TestStatus_JSONRoundTrip`
- `internal/shedengine/doc.go`

**Intent:**
Record who moved a halted run on, so the timing report (card 10) can tell the orch, the LLM driver, the operator and the Go driver apart.
`shedengine` takes the label as a plain string and gains no import, as `PATTERN-shed-producer-seam` requires; who the caller is gets decided in the CLI layer (card 6).

`HistoryEntry` gains `By string` with `json:"by,omitempty"`, the caller label of the goto or resume that wrote the entry.
`Status` gains `PendingBy string` with `json:"pending_by,omitempty"`, the label of a step that resumed a halted run, waiting for the next verdict to carry it.
Both are tolerant: a record without them decodes with empty values and re-encodes without the keys, which `TestStatus_JSONRoundTrip` gains a row for.
`CallerShed` is the label the Go driver and batten's automatic steps pass, and the one a report reads as class `shed`.

Goto and relaunch.
`GotoRequest` gains `By string`; `Goto` writes it on the appended `OutcomeGoto` entry, omitted when empty, and clears `PendingBy`, since the goto is what moved the run on.
`TestGoto_MovesBlockedRunToTarget` sets `By: "orch"` on a status carrying a pending label and asserts the goto entry's `By` is `orch` and `PendingBy` is empty, and gains a row with an empty `By` asserting the written entry has no `by` key.
`Relaunch`, the arbiter's and tackler's move, also clears `PendingBy` and writes no `by` on its goto entry, since those settles are read from their own records; `TestRelaunch` gains a row starting from a pending label and asserting it cleared.

Resume.
`Shed` gains `ResumedBy string`, the told label of the caller stepping it, empty for none.
`Shed.stepLocked`'s resume write, which runs under the run lock only when the status read is not running, sets `PendingBy` to `ResumedBy` only when the status it read is blocked, failed or awaiting, `ResumedBy` is non-empty and no label is pending, matching `RecordResumer`, so a resume from paused records nothing and a step refused as busy, or never built, records nothing.
`RecordResumer` sets `PendingBy` to `by` under the status lock when the status file's state is blocked, failed or awaiting, `by` is non-empty and no label is pending already, for the loop's waiter, which records before its children step; it takes no run lock, writes nothing in any other case, returns nil for an absent status file, since the step that follows refuses that itself, and returns the read error for a status file that does not decode.
In both, the first label stands while the run's loop children, each with its own label, step the run on.
`Shed.persistTransient` hands the label on: when the write appends a history entry, the appended entry takes `PendingBy` and `PendingBy` clears; a write that appends none, the resume write itself or a fold into the last entry, carries the pending label through, so a label never lands on an entry written before the resume.
`TestStep_ResumeFromHaltedStates` steps each halted state with `ResumedBy` `orch` and asserts the appended entry's `By` is `orch` and `PendingBy` empty for blocked, failed and awaiting; a row resuming from paused with `ResumedBy` `orch` asserts the appended entry carries no `by` and `PendingBy` stays empty; a row whose producer returns a budget-exempt Stuck folding into the last entry asserts the folded entry gains no `By` and `PendingBy` stays `orch`; a row with an empty `ResumedBy` writes no `by`.
`TestRecordResumer` in `internal/shedengine/status_test.go` covers the waiter's marking: blocked, failed and awaiting runs are marked; running and paused runs are left unmarked; a second `RecordResumer` with `driver` over a pending `orch` keeps `orch`; an empty label marks nothing; an absent status file returns nil and creates none; an undecodable one returns an error.
`internal/shedengine` holds 140 top-level tests against its row of 141, so `TestRecordResumer` needs no budget change.

Bound: the label is for reports only; no read of `By`, `PendingBy` or `ResumedBy` enters any Go decision, and the refusals of goto and step are unchanged.
A waiter whose loop then fails to spawn leaves its label pending until the next goto, relaunch or appended entry, and that entry carries it; the report's settler is then that waiter's caller.
`internal/shedengine/doc.go` gains a paragraph on the fields, who writes them, the first-label-stands rule and that no decision reads them.

**ImpactSummary:** two optional status-file fields, one Shed field and one GotoRequest field; GotoRequest's one constructor in shedverbs compiles unchanged until card 6 fills By, Relaunch's callers are untouched, and every older record decodes as before.

**Commit:** 5: Record the caller label on goto entries and resumed history entries
