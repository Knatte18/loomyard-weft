# Card 21 — turn-end-counts-with-nothing-outstanding

**Edit:**
- `internal/websterengine#TurnEndRead`
- `internal/websterengine#TurnEndedAfter`
- `internal/websterengine#awaitTerminal`
- `internal/websterengine/recoverbatch_test.go`
- `internal/websterengine/doc.go`

**Intent:** Take the shell expiry out of webster's turn-end reading, so a recovery strand waiting on a shell of either signal runs until its report exists or `recovery_timeout_min` (D9).
`TurnEndedAfter` counts a turn end only with nothing outstanding, once it has stood for the settle; a turn end left waiting on any task, a shell of either signal, a fork or an awaited shell, does not count, so it no longer calls `ShellWaitExpires`.
`TurnEndRead` drops its `ShellWait` field, and `awaitTerminal` stops passing the shell bound; `Classify` reads the report before any turn end, as today.
`doc.go`'s recovery passage says the strand's turn end counts only with nothing outstanding and names the timeout as the bound.
`TestRecoverBatch_SecondCall` already drives `TurnEndedAfter` through `RecoverBatch` over real Claude hook lines with the signal-parsing engine, so it carries the acceptance rows and no new test is added.
Its row `a turn end waiting on a transcript-reported shell is dead once background_shell_wait_min has passed` is reversed: the same transcript-reported shell, outstanding past the bound, keeps the strand running, checked with `checkRecoveryRunning`, and the row's name and comment say so.
Its payload-reported shell row, its plain turn end row and its row whose next turn starts after the turn end already pin a payload shell not counting, a turn end with nothing outstanding counting after the settle and a later turn start not counting, and stay as they are.
It gains one row: a turn end with a shell outstanding and the recovery's report present classifies done from the report.
The test's doc comment drops the background-shell bound from its turn-end sentence.

**ImpactSummary:** One reader and its one caller in websterengine; shuttleengine's exported bound and expiry rule are left for card 22 to remove, so the build stays green between the two.
