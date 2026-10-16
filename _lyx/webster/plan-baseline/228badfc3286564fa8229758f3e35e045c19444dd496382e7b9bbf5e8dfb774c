# Card 11 — creel-output-consistency

**Edit:**
- `internal/creelengine#LogEntry`
- `internal/creelengine#pool.logEntryFor`
- `internal/creelengine#daemon.dispatch`
- `internal/creelengine#daemon.awaitOutcome` -> `func (d *daemon) awaitOutcome(conn net.Conn, reader *bufio.Reader, waiter *Waiter, id int64, sender Sender) error`
- `internal/creelengine#Store.Release`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestStore_InjectAndHold`
- `internal/creelengine_test#TestScheduler_Reminders`
- `internal/creelcli#logEntryFields`
- `internal/creelcli#injectCmd`
- `internal/creelcli#interruptCmd`
- `internal/creelcli#holdCmd`
- `internal/creelcli#releaseCmd`
- `internal/creelcli#TestCreelCLI`

**Uses:**
- `internal/creelengine#Store.strandTarget`
- `internal/creelengine#Store.liveHoldLocked`
- `internal/creelengine_test#env`
- `internal/creelengine_test#newEnv`
- `internal/creelengine_test#program`
- `internal/creelengine_test#external`
- `internal/creelengine_test#wantErr`
- `internal/creelengine_test#eventsOf`
- `internal/testkit/creelfake#LiveStrand`
- `internal/creelcli#rig.startDaemon`
- `internal/creelcli#runCreel`

**Intent:** Decision D8: `log`, the help texts, the waiting verbs' sender and `release` say what `doc.go` says.

- `log` prints a `reminded` record with its message ids and its backoff step: `LogEntry` gains `Unread []int64` and `Step int`, `pool.logEntryFor` copies them from the record, and `logEntryFields` prints `unread` and `step` when set and leaves out `id` when it is 0, so no event prints `"id":0`.
- Every creel help text that states the privilege rule states it as `doc.go` does: EXTERNAL callers, programs and orch strands may inject, interrupt and hold on any strand, and any other strand on its own strand only.
  `injectCmd`'s and `interruptCmd`'s `Long` replace their "Only the orch, a program and an external caller may …" sentence with that rule, and `holdCmd`'s and `releaseCmd`'s `Long` gain it, `release` adding that an unknown name is refused with the names that exist.
- Every verb that prints a sender prints the resolved one: `daemon.awaitOutcome` takes the resolved sender from `daemon.dispatch` and sets `Sender` on its pending and final responses, so `post --wait-reply` and `inject --wait` print the sender the daemon resolved instead of an empty one.
- `Store.Release` first ends the caller's live hold on the name, so a hold on a strand removed since can still be released; another sender's live hold is refused as today; with no live hold, a name that is no existing strand is refused with `strandTarget`'s error naming the strands that exist, as `hold` is, and an existing strand the caller does not hold answers `released:false` as today.
  The strand listing is read outside the store's mutex, after the hold check, as `strandTarget` already reads it.

Tests:
`TestStore_InjectAndHold`'s `Release_WithNoHoldChangesNothing` row keeps its no-hold answer, and new rows pin that `Release` of an unknown name is refused naming the existing strands and journals nothing, that the owner's hold on a strand the fake delivery no longer lists is released with a `released` record, and that with no live hold a strand listing that fails returns that read error and journals nothing.
`TestScheduler_Reminders`'s `UnansweredReminderBacksOffToTheCeilingAndANewMessageResets` row, which already drives successive reminders through their steps, also asserts each `reminded` log entry carries the reminded ids and its backoff step.
`TestCreelCLI` gains rows: `creel log` after a reminder prints its `reminded` event with `unread` and `step` and no `id`; and as EXTERNAL, `inject --wait` and `post --wait-reply` print `sender` as the hub's EXTERNAL name.

`internal/creelengine/doc.go`'s Holds paragraph says release ends the caller's hold even on a removed strand and refuses a name that neither holds nor exists.

**ImpactSummary:** Touches log rendering, the daemon's wait answer (one re-signed unexported method, its callers all in dispatch) and Release's refusal; no exit code or field of another verb changes.
