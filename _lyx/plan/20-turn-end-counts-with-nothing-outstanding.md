# Card 20 — turn-end-counts-with-nothing-outstanding

**Edit:**
- `internal/websterengine#TurnEndRead`
- `internal/websterengine#TurnEndedAfter`
- `internal/websterengine#awaitTerminal`
- `internal/websterengine/strand_test.go`
- `internal/websterengine/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:** Take the shell expiry out of webster's turn-end reading, so a recovery strand waiting on a shell of either signal runs until its report exists or `recovery_timeout_min` (D9).
`TurnEndedAfter` counts a turn end only with nothing outstanding, once it has stood for the settle; a turn end left waiting on any task, a shell of either signal, a fork or an awaited shell, does not count, so it no longer calls `ShellWaitExpires`.
`TurnEndRead` drops its `ShellWait` field, and `awaitTerminal` stops passing the shell bound; `Classify` reads the report before any turn end, as today.
`doc.go`'s recovery passage says the strand's turn end counts only with nothing outstanding and names the timeout as the bound.
`TestTurnEnded` drives an engine without the session-signal capability, where no shell is ever outstanding, so a new `TestTurnEndedAfter_OutstandingWork` over a test-local engine implementing `shuttleengine.SessionSignalParser` pins the acceptance rows: a turn end with a transcript-reported shell outstanding does not count, one with a payload-reported shell does not count, one with nothing outstanding counts after the settle, and a later turn start does not count.
The websterengine row of `test-budget.yaml` rises by one.

**ImpactSummary:** One reader and its one caller in websterengine; shuttleengine's exported bound and expiry rule are left for card 21 to remove, so the build stays green between the two.
