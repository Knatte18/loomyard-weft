# Card 15 — batten-shadow-log

**Edit:**
- `internal/battenshed/deps.go`
- `internal/battenshed/notice.go`
- `internal/battenshed/notice_test.go`
- `internal/battencli/wire.go`
- `internal/battencli/wire_seams_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#ReadSessionStates`

**Intent:**
Log each live run's session state where batten's activity reading decides a quiet or API-error notice (D5, third bullet), in shadow mode.

`battenshed.AgentActivity` gains the run's session state as neutral strings: the state, its cause and its since, empty when unread.
`battencli`'s `readAgentActivity` reads `ReadSessionStates` over the same config, anchor and Claude engine as `ReadAgentActivity`, with `time.Now()`, and fills each reading's state from the entry with the same strand name.
A failed state read is logged at Warn once per read and leaves the state fields empty; the activity reading, `waitLive` and the returned error are what they would be without it.
`judgeAgents` logs `battenshed: session state at notice` at Info, once per run, when it returns a quiet or API-error finding, naming the producer, the condition, and the run's state, cause and since, so each notice can be checked against the state; it decides nothing on them.

Bound: no notice, stamp, finding or return depends on the state fields.

Tests:
- `notice_test.go`: one new table-driven top-level function, since no existing notice test captures logs: with log capture, a quiet finding and an API-error finding each log the state of every run, and a row with the state fields set yields the same findings and notifier calls as the same row with them empty.
- `wire_seams_test.go`: `TestReadAgentActivity_ReadsLiveRunsAndTheWaitMarker` asserts that the live run's reading carries the state of its stamped turn end, and its `LastActivity`, `APIError` and `waitLive` are the values the existing row expects.
- `cmd/lyx/testdata/test-budget.yaml`: the `internal/battenshed` row rises by 1, for the new `notice_test.go` function.

**ImpactSummary:** Batten's agent-activity reading carries and logs each run's session state; its notice decisions are unchanged.
