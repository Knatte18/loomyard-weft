# Card 2 — drift-records

**Create:**
- `plan:internal/driftengine#ScopeRecord` -> `type ScopeRecord struct`
- `plan:internal/driftengine#Acceptance` -> `type Acceptance struct`
- `plan:internal/driftengine#CaptureRecord` -> `type CaptureRecord struct`
- `plan:internal/driftengine#ReadScope` -> `func ReadScope(p Paths) (ScopeRecord, bool, error)`
- `plan:internal/driftengine#UpdateScope` -> `func UpdateScope(p Paths, mutate func(cur ScopeRecord, found bool) (ScopeRecord, error)) error`
- `plan:internal/driftengine#ReadCapture` -> `func ReadCapture(p Paths) (CaptureRecord, bool, error)`
- `plan:internal/driftengine#WriteCapture` -> `func WriteCapture(p Paths, rec CaptureRecord) error`
- `plan:internal/driftengine#CountedCapture` -> `func CountedCapture(scope ScopeRecord, capture CaptureRecord, found bool, awaitRow bool) bool`
- `plan:internal/driftengine#ErrRunsSpent` -> `var ErrRunsSpent = errors.New("drift: the run budget is spent")`
- `plan:internal/driftengine#TakeRun` -> `func TakeRun(p Paths, max int) (int, error)`
- `plan:internal/driftengine#ResetRuns` -> `func ResetRuns(p Paths, head string) error`
- `plan:internal/driftengine#AttemptHead` -> `func AttemptHead(p Paths) (string, bool, error)`
- `plan:internal/driftengine#CloseAgentSession` -> `func CloseAgentSession(p Paths) error`
- `plan:internal/driftengine#AgentSessionOpen` -> `func AgentSessionOpen(p Paths) (bool, error)`
- `plan:internal/driftengine#TestCountedCapture` -> `func TestCountedCapture(t *testing.T)`
- `plan:internal/driftengine#TestTakeRun` -> `func TestTakeRun(t *testing.T)`
- `plan:internal/driftengine#ScopeRecord.Unruled` -> `func (s ScopeRecord) Unruled() []Acceptance`

**Extends:**
- `internal/driftengine/doc.go`

**Uses:**
- `plan:internal/driftengine#Paths`

**Intent:**
Add the Go-owned records every Drift row and verb reads and writes, all through `internal/state` so an unknown key survives a rewrite (PATTERN-tolerant-records).

`ScopeRecord` (`_lyx/drift/scope.json`, durable, D3/D5/D6/D7) holds:
`scope` (`off`, `partial` or `full`, as named constants `ScopeOff`, `ScopePartial`, `ScopeFull`), `skip` (the `--skip` tests), `recorded_at`;
`awaited_at` and `deferred` (D5: set by a passed `Drift-Await` or by `lyx drift defer`, the latter with `deferred: true`);
`waived_reason` and `waived_at` (D6);
`accepted`, a list of `Acceptance` (`test`, `reason`, `by` — `agent` or `caller` — `ruling` — `unruled`, `ruled` or empty under `partial` — and `at`);
`withdrawn`, the tests moved off `accepted` (D7);
`halt_named` and `halt_named_at`, the `unruled` tests a `Drift-Verify` decision halt last named and when (D6);
`gate_halt_at`, the time of the dirty-tree or test-gate Stuck `Drift-Verify` last returned, which its resumed entry compares the run status's history against, since the status keeps no Stuck reason a resume can read (D6).
`halt_named_at` and `gate_halt_at`, the two the history test reads, are strings `Drift-Verify` formats `Deps.Now().UTC().Format(time.RFC3339)`, the format `shedengine`'s `nowRFC3339` gives `HistoryEntry.At` and `LastAt`;
the history test of card 11-drift-verify-producer parses both sides and compares them as instants at second precision, so a record written in the same second as the shed's second-truncated stamp still reads as earlier or equal.
`Unruled` returns the accepted entries whose ruling is `unruled`.

`CaptureRecord` (`.lyx/drift/capture.json`, D4) holds `state` (`running`, `passed`, `failed` as constants), `pid` (the capture process, so a spawner tells its own child's record from an older one), `head`, `slot_acquired_at`, `command_started_at`, `ended_at`, `exit_code`, `reason` and `log`.
`ReadScope`/`ReadCapture` answer found false for an absent file; `UpdateScope` is `state.UpdateJSON` under the record's own lock file; `WriteCapture` is `state.WriteJSON`.

`CountedCapture` is D5's counted-capture predicate: a found capture in state `passed` and, when `awaitRow` (the loom recipe, which has `Drift-Await`), `awaited_at` set in the scope record; in darn (`awaitRow` false) a `passed` record alone.

`TakeRun` counts one compare or side-capture run of a `Drift-Verify` agent session in `.lyx/drift/runs.json`: below `max` it increments and returns the new count; at `max` it refuses with an error wrapping `ErrRunsSpent` and naming the budget, and writes nothing.
Its caller words the way forward for who called it (card 7-drift-verbs-cli): the agent session's is `write what you found to verify-report.md and end your turn, which stops the row` (D6).
`ResetRuns` writes the count back to zero, records HEAD at the attempt's start, which `AttemptHead` reads back (found false with no record), and opens the agent-session mark; `Drift-Verify` calls it at each fresh spawn of its agent, so each attempt has the whole budget and its verify gate can tell whether the session committed code (card 11-drift-verify-producer).
`CloseAgentSession` closes the mark and keeps the count and the attempt head; `AgentSessionOpen` reads it, false with no record; a verb reads it to tell the agent session from the row's own mechanical work (Shared Decision "run status decides the caller").

`doc.go` gains the records section.

Tests, tier0: `TestCountedCapture` is a table over found/absent, each state, `awaited_at` set or not and `awaitRow` true or false; `TestTakeRun` takes runs up to the budget, asserts the refusal wraps `ErrRunsSpent` and that the count stays, then `ResetRuns` admits again and `AttemptHead` returns the head it recorded; `AgentSessionOpen` reads false with no record, true after `ResetRuns`, and false after `CloseAgentSession`, which leaves the count and `AttemptHead` as they were.
`Unruled` is reached by `TestDriftVerify`'s full-scope rows (card 11-drift-verify-producer), which assert the named tests.

**ImpactSummary:** Only the new package's doc grows; no existing code changes.
