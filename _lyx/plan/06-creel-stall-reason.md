# Card 6 — creel-stall-reason

**Create:**
- `internal/creelengine#Idleness`
- `internal/creelengine#Stall`
- `internal/creelengine#StallTooShort`
- `internal/creelengine#StallDraft`
- `internal/creelengine#Stall.Reason`
- `internal/creelengine#eventStalled`
- `internal/creelengine#Store.noteStall`
- `internal/testkit/creelfake#Delivery.SetStall`

**Edit:**
- `internal/creelengine#Delivery`
- `internal/creelengine#worker.idle`
- `internal/creelengine#worker.typeInject`
- `internal/creelengine#worker.remind`
- `internal/creelengine#Store`
- `internal/creelengine#OpenStore`
- `internal/creelengine#record`
- `internal/creelengine#LogEntry`
- `internal/creelengine#pool.apply`
- `internal/creelengine#pool.logEntryFor`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/creelengine_test#TestScheduler_Reminders`
- `internal/testkit/creelfake#Delivery`
- `internal/testkit/creelfake#Delivery.Idle` -> `func (d *Delivery) Idle(strand creelengine.Strand) (creelengine.Idleness, error)`
- `internal/creelcli#hubDelivery.Idle` -> `func (d *hubDelivery) Idle(strand creelengine.Strand) (creelengine.Idleness, error)`
- `internal/creelcli#logEntryFields`
- `internal/creelcli#stubOps`
- `internal/creelcli#stubOps.SessionIdle`
- `internal/creelcli#TestHubDelivery`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli#TestSmokeCreelClaudeStrand`

**Uses:**
- `internal/shuttleengine#IdleProbe`
- `internal/creelengine_test#env`
- `internal/creelengine_test#newEnv`
- `internal/creelengine_test#env.start`
- `internal/creelengine_test#env.tick`
- `internal/creelengine_test#env.probeWaits`
- `internal/creelengine_test#eventsOf`
- `internal/creelengine_test#awaitOutcome`
- `internal/creelcli#rig.startDaemon`
- `internal/creelcli#runCreel`

**Intent:** Decision D2, engine half: the Delivery seam's idle answer carries a stall reason, the engine journals each change of it as a `stalled` record, and `log` prints it.

`Delivery.Idle` now answers `(Idleness, error)`: `Idleness` holds `Idle bool` and `Stall Stall`, and `Stall` is empty on an idle answer and on any not-idle reading that names no reason (a running turn, a dialog, an operator active, the pane probe's not-idle).
The two reasons are `StallTooShort` and `StallDraft`, and `Stall.Reason` renders each for a reader:
too short reads "the pane is too short to show the input box; way forward: give the strand's pane more rows, as `lyx reed attach` from a taller terminal does", a real `lyx` command no agent's settings deny (`PATTERN-no-denied-recovery`), worded so it can stand in for the orch watcher's `paneTooShortReason` once `creel-notices` drops that probe;
draft reads "the input box holds a draft; the delivery waits until it is sent or cleared".
The seam's doc comment gives the shape; `PATTERN-creel-store` holds, since only the seam's answer changes, not its route.

`internal/creelcli`'s `hubDelivery.Idle` maps shuttle's probe: `TooShort` gives `StallTooShort`, else `Draft` gives `StallDraft`, else none; an idle probe carries none.
The fake: `creelfake.Delivery` keeps a scripted stall per strand, `SetStall` sets it (empty clears), and `Delivery.Idle` answers `{Idle: false, Stall: s}` for a strand with a scripted stall, else as today; the kit's package doc names the scripted stall beside the idle and pause flags.

The engine: `Store` keeps, per receiver, the last stall it was told with the clock time it was read, initialised empty in `OpenStore`.
`Store.noteStall(receiver, stall)` journals `{event: stalled, to: receiver, reason: <stall>}` only when stall is non-empty and differs from the receiver's last one, and records the new stall and time; an empty stall clears the entry with no record.
`worker.idle` passes every reading to `noteStall`, an error counting as no reason, so a probe that reads the same reason again journals nothing; `worker.typeInject` and `worker.remind` clear the stall once a send went out, a delivery ending the episode with no record.
The worker only probes while something is pending, so a stall is journaled only while something is pending.
Bound: one journal line per change of reason per receiver, logged nowhere per probe (`PATTERN-log-levels`), so an operator typing a draft adds at most one line per typing episode while something is pending.
The TTL keeps running while a stalled strand is live and unpaused, as `doc.go` states today; nothing in the aging changes.

The journal: `record` gains `Reason string` (`json:"reason,omitempty"`), and `LogEntry` gains `Reason`, filled by `pool.logEntryFor`; `pool.apply` takes `eventStalled` as a log event that changes no pool state.
An older binary's fold ignores the unknown event name as it ignores any (`PATTERN-tolerant-records`).
`creelcli`'s `logEntryFields` prints `reason` when it is set, so `log` shows a `stalled` record with its receiver and reason.

Tests:
`TestScheduler_Injects` gains a row where a receiver scripted `StallDraft` across probes, then `StallTooShort`, then idle, journals exactly two `stalled` records with those reasons and to that receiver, and none once the inject is typed;
a row where a receiver reads `StallDraft`, then not idle with no stall, then `StallDraft`, then an `Idle` error, then `StallDraft` again journals three `stalled` records, so a reading with no reason and an error each clear the stall without a record of their own;
and a row where a receiver stalled for its whole TTL still expires its inject at the TTL with one `stalled` record, so the TTL runs while stalled.
`TestScheduler_Reminders` gains a row where a receiver with unread mail reads `StallDraft`, then idle so its reminder is typed, then `StallDraft` again, journaling two `stalled` records, so a typed reminder clears the stall.
`TestHubDelivery`'s Idle row checks `{TooShort}` maps to `StallTooShort`, `{Draft}` to `StallDraft`, both set to `StallTooShort`, `{Idle}` to an idle answer with no stall, and `{}` to not idle with no stall, and `stubOps` gains `tooShort` and `draft` fields its `SessionIdle` answers.
`TestCreelCLI` gains a row where an inject to a strand the fake delivery reads as `StallTooShort` shows, in `creel log`, one `stalled` event with `to` the strand and the too-short `reason`.
`TestSmokeCreelClaudeStrand` reads `Idle` off the new answer.

`internal/creelengine/doc.go`'s Layering and Delivery loop paragraphs say the idle answer carries the stall reason and the engine journals each change as `stalled`, and its Log paragraph names the `stalled` record.

**ImpactSummary:** Re-shapes the one Delivery method whose implementers are creelfake.Delivery and creelcli's hubDelivery, both re-signed here, plus their two test callers; the journal gains one tolerated event.

**Verify:** lyx gate test --tags llm ./internal/creelcli
