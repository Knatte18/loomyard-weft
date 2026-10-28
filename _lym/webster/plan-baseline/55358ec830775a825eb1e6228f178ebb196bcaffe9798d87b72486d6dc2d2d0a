# Card 7 — chase-clock

**Create:**
- `internal/creelengine/chase.go`
- `internal/creelengine#chaseInterval`
- `internal/creelengine#maxChases`
- `internal/creelengine#chaseProbeCeiling`
- `internal/creelengine#ChaserProgram`
- `internal/creelengine#eventChased`
- `internal/creelengine#eventChaseAged`
- `internal/creelengine#eventUnanswered`
- `internal/creelengine#UnansweredNoReply`
- `internal/creelengine#UnansweredHolderGone`
- `internal/creelengine#OutcomeUnanswered`
- `internal/creelengine#isChaseable`
- `internal/creelengine#chaser`
- `internal/creelengine#chaser.run`
- `internal/creelengine#Store.chaseSubjects`
- `internal/creelengine#Store.chasePending`
- `internal/creelengine#Store.ageChases`
- `internal/creelengine#Store.persistChaseAged`
- `internal/creelengine#Store.stepChases`
- `internal/creelengine#Store.settleUnansweredLocked`
- `internal/creelengine#Store.repairChases`
- `internal/creelengine_test#isChaseWait`
- `internal/creelengine_test#env.writeJournal`
- `internal/creelengine_test#env.reopen`
- `internal/creelengine_test#TestScheduler_Chases`

**Edit:**
- `internal/creelengine#record`
- `internal/creelengine#Message`
- `internal/creelengine#LogEntry`
- `internal/creelengine#pool`
- `internal/creelengine#newPool`
- `internal/creelengine#pool.apply`
- `internal/creelengine#pool.applyPosted`
- `internal/creelengine#pool.applyRead`
- `internal/creelengine#pool.applyCarried`
- `internal/creelengine#pool.carryRecords`
- `internal/creelengine#pool.moveOrigins`
- `internal/creelengine#pool.logEntryFor`
- `internal/creelengine#Store`
- `internal/creelengine#OpenStore`
- `internal/creelengine#Store.settledOutcomeLocked`
- `internal/creelengine#Reading.Ack`
- `internal/creelengine#Store.markNoticeRead`
- `internal/creelengine#daemon.awaitOutcome`
- `internal/creelengine#Scheduler`
- `internal/creelengine#NewScheduler`
- `internal/creelengine#Scheduler.Start`
- `internal/creelengine#Scheduler.Stop`
- `internal/creelengine#Scheduler.wake`
- `internal/creelengine#TestJournal_Fold`
- `internal/creelengine#TestJournal_Retention`
- `internal/creelengine_test#isSweepWait`
- `internal/creelengine/doc.go`

**Extends:**
- `pattern/PATTERN-daemon-wakeups.md`
- `cmd/lym#unboundedWaitFiles`
- `docs/overview.md`

**Uses:**
- `internal/creelengine#Store.appendPostedLocked`
- `internal/creelengine#Store.deliverMailLocked`
- `internal/creelengine#Store.answerLocked`
- `internal/creelengine#Store.liveHoldLocked`
- `internal/creelengine#agedPersistInterval`
- `internal/creelengine_test#env`
- `internal/creelengine_test#newEnv`
- `internal/creelengine_test#env.start`
- `internal/creelengine_test#awaitOutcome`
- `internal/agentname#FormatProgram`
- `internal/agentname#ParseProgram`

**Intent:**
Creel chases a marked message its holder read and did not answer (D10), journals each chase and the outcome (D11), and answers a waiting `--wait-reply` with `unanswered`.

A message is chaseable (`isChaseable`) when it carries the mark, is read, is not replied, has no `unanswered` record, and its holder is a strand; a chase carries no mark, so it is never chaseable itself.
A marked message never read is not chased, and neither is one a program holds.

Journal and fold:
- `record` gains `Chase int` (`json:"chase,omitempty"`), the chase number on a chase's `posted` record and on a `chased` record, `Chased int64` (`json:"chased,omitempty"`), the chased message's id on the chase's `posted` record, `ReadAt time.Time` (`json:"read_at,omitzero"`) and `MarkSeconds int64` (`json:"mark_s,omitempty"`) on a carried snapshot; `ConsumedSeconds` carries the chase clock on a `chase-aged` record, on a chase's `posted` record (the clock at that step) and on a carried snapshot.
- `Message` gains `ReadAt`, `ChaseConsumed` and `ChaseMark` (`time.Duration`, the clock at the last step, 0 before one), `Chases int`, `Unanswered string` (the reason, empty while none) and, on a chase, `ChaseOf int64`.
- `applyRead` stamps `ReadAt`, and a read by a program records it in a new `pool` map `readers`, built in `newPool`, from each program that has read to the journal day of the read record that put it there; `applyPosted` leaves `readers` alone, so a program that only posted is not in it, whatever the `programs` map of known mailboxes holds.
- `applyPosted` leaves `programs` alone for every `posted` record whose sender's program name, read with `agentname.ParseProgram`, is `ChaserProgram`, a chase and a sender notice alike, and `applyCarried` leaves `programs` alone for a carried `posted` snapshot whose `From`, read with `agentname.ParseProgram`, is `ChaserProgram`, exactly as `applyPosted` does, so `CREEL` never becomes a receiver, whether the record is an original line or a retention snapshot.
  A `posted` record with `Chased` creates the chase message with `ChaseOf` and raises the chased message's `Chases` and `ChaseMark` to its `Chase` and `ConsumedSeconds`; `chased` records the highest chase number per message in a second new `pool` map; `chase-aged` sets `ChaseConsumed`; `unanswered` sets `Unanswered` from the reason.
  `chased`, `chase-aged` and `unanswered` are log events; `LogEntry` gains `Chase int`, set on a `chased` entry, and `unanswered` carries its reason in `Reason`, so `lym creel log --id <id>` lists a chased message's mark, reads, chases and outcome.
- Retention carries a chaseable message forward like an unread one: `carryRecords` writes its snapshot with `ReadAt`, `ConsumedSeconds`, `Chase` (the chase count) and `MarkSeconds`, `applyCarried` folds a snapshot with `ReadAt` as read with those values, and `moveOrigins` moves its origin, so `prune` keeps it.
  A carried snapshot of a program's read keeps that program in `readers`: `carryRecords` writes a carried read record for each reader whose day is before the cutoff, `applyCarried` folds that record back into `readers`, and `moveOrigins` moves the reader's day as it does a message's origin.
- `OpenStore` runs `repairChases` after the fold: a chase `posted` record whose chase number has no `chased` record gets that `chased` record written, so a crash between the chase's post and its `chased` record never yields a third chase.

The clock (`internal/creelengine/chase.go`):
The `Scheduler` keeps one `chaser` per holder of a chaseable message, started by `Start` for every holder `chaseSubjects` lists and by `wake` when `chasePending(holder)` holds and none runs; `Reading.Ack` and `markNoticeRead` signal the holder's wake when they journal a read of a marked message, which also wakes the removed-holder sweep.
`chaser.run` probes the holder once when it starts and counts that reading as the previous tick's, then ticks every `agedPersistInterval` while the clock runs, so the first tick, one `agedPersistInterval` after the start, charges when both readings find the clock running: each tick probes the holder through the delivery seam, and the clock runs while the holder is listed, live and not covered by a pause.
A tick that reads the clock stopped doubles the next wait, up to `chaseProbeCeiling`, since the probe runs `reed status` and so tmux (`PATTERN-daemon-wakeups`' Backoff rule); a tick that reads it running, or a wake of the holder, returns the wait to `agedPersistInterval`.
A tick whose probe and the previous tick's both read the clock running, one `agedPersistInterval` apart, charges one whole interval through `ageChases`, which adds it to each chaseable message of the holder that no other sender's live hold stands on (a hold whose owner is the message's sender lets the clock run); a backed-off wait therefore never charges stopped time.
A wake between running ticks only makes the chaser re-read which messages it chases; it keeps the tick's deadline.
A wake during a backed-off wait starts a fresh `agedPersistInterval` wait from the wake.
A holder coming back live fires no wake, so a stopped clock's restart is seen at the next backed-off tick, up to `chaseProbeCeiling` later, and the running time before that tick and the one interval after it goes uncharged; a chase can therefore land up to that much later than its interval of running time.
So every chaser wait lies between `agedPersistInterval` and `chaseProbeCeiling`, the one-second floor holds, and the clock counts whole minutes of running time.
At each tick `persistChaseAged` journals a `chase-aged` record for each message whose consumed seconds changed, and it does so again when the clock stops and when `Scheduler.Stop` stops the chaser, as an inject's consumed TTL is persisted.
`stepChases` takes the steps that are due: a step is due when `ChaseConsumed` reaches the step's base plus `chaseInterval`, the base being `ChaseMark`, or, for a message whose step was already due when the store opened, the clock at that opening, so after downtime the overdue steps go one at a time, each one interval of running time after the later of the previous step and the restart.
Steps one and two are chases; the step after the second chase settles the message `unanswered` with `UnansweredNoReply`.
A step therefore lands at the tick that completes its interval: with the clock running from the chaser's start, chase 1 lands at the 15th tick after the start, and a step one interval after a restart lands at the 15th tick after the restart, the tick that completes that interval; the chaser ends when the holder holds nothing chaseable.

A chase is a creel post signed `<shortname>:CREEL` (`agentname.FormatProgram` with `ChaserProgram`) to the holder, through `appendPostedLocked` with `Chase`, `Chased` and `ConsumedSeconds` set, then its `chased` record and `deliverMailLocked`.
Its body names the message id, the message's sender name (an EXTERNAL sender by its bare name, never its envelope) and the reply command `lym creel post --reply <id> "<text>"`, and carries no text the sender wrote, so it cannot pose as an instruction; it rides the ordinary delivery, hooks for a hook strand and a typed notice for a pane-mail strand.
`settleUnansweredLocked` journals the `unanswered` record (id, holder, reason), then posts the sender a `CREEL` notice naming the message, the receiver and the way forward, `lym creel log --id <id>` for its path, then ask again or decide without the answer, when the sender is a strand the delivery seam lists or a program in `readers`; any other sender, EXTERNAL, a program that never read and a strand no longer listed, gets no notice and the journal record alone stands, and a notice that fails to store is logged and never undoes the record.
It then answers every wait on the message's reply with `OutcomeUnanswered` carrying the message, whatever became of the notice.
`settledOutcomeLocked` answers a reply wait registered after the record with `OutcomeUnanswered` too, which is what a reconnecting `--wait-reply` registers, and `daemon.awaitOutcome` puts the message in the response for that outcome, as it does for a reply or a return.
A reply that comes later is still delivered.
Bound: at most two chase posts and one sender notice per marked message.
`PATTERN-daemon-wakeups.md` lists the chase timer among creel's waits, its bullet stating one `agedPersistInterval` wait while the clock runs and a wait doubling to `chaseProbeCeiling` while it is stopped, back to the base on a running tick or a wake of the holder, and `cmd/lym#unboundedWaitFiles` gains `internal/creelengine/chase.go`.

Tests, all over an injected clock:
`internal/creelengine_test#isSweepWait` narrows to `8s < d <= 50s`, the sweep ceiling, and the new `isChaseWait` matches `agedPersistInterval <= d <= chaseProbeCeiling`, the chaser's waits, so the sweep, probe and chase waits stay apart on the shared fake clock.
`env.writeJournal` writes day files under the env's journal directory before a store opens, and `env.reopen` stops the env's scheduler, closes its store and opens a new store and scheduler over the same journal.
`TestScheduler_Chases` is new in `deliver_test.go` (`//go:build tier0`), over `newEnv`'s store and a started scheduler, the clock moved one chase wait at a time.
Rows:
- a marked message from `batten`, whose mailbox a read made known, posted with a wait to the pane-mail strand `alpha`, typed to it as a notice and so read through `markNoticeRead`, gets chase 1 at the 15th tick after its chaser starts, chase 2 at the 30th and is `unanswered` at the 45th, with a sender notice to `batten` naming `lym creel log --id` and `OutcomeUnanswered` carrying the message on the wait;
- a marked message from the strand `beta` that `alpha` reads through `Read` and `Ack` is chased the same way, and `beta` gets the notice;
- a reply before the first chase stops every step, one after the first chase stops the second, and a reply after the `unanswered` outcome is still delivered to the sender;
- a `WaitReply` registered after the outcome is answered `OutcomeUnanswered` at once;
- an unread marked message is never chased, and neither is a marked message held by a program;
- while the clock runs, a `chase-aged` record carrying the consumed seconds is journaled at each tick, and a pause covering the holder stops the clock, as do a dormant holder and a hold by another sender, while a hold by the message's own sender does not;
- a dormant holder's chaser waits double from one minute to `chaseProbeCeiling` and drop back to one minute once the holder is live again, and no time is charged meanwhile;
- `env.reopen` after a pause counts no paused time and neither repeats nor skips a chase;
- a journal `env.writeJournal` seeds with a chase `posted` record and no `chased` record gets the record written at open and yields no third chase;
- a seeded journal whose consumed clock passed the next step before the restart takes that step at the 15th tick after the restart and the next at the 30th;
- a program sender that posted but never read gets no notice while the journal records the outcome, and so does a strand sender removed from the fake delivery before the outcome, whose wait is still answered;
- a chase of an EXTERNAL sender's message names the bare EXTERNAL name and no envelope, carries none of the sender's text, and its outcome posts no notice.

`TestJournal_Fold` gains rows folding the chase records into the message's chase count, mark, consumed clock and reason, a program's post leaving it out of `readers` while its read puts it in, and a sender notice folded without making `CREEL` a known mailbox; `TestJournal_Retention` gains a row carrying a read, marked, unanswered message with its consumed clock past retention, which also asserts that a program whose only read is past retention is still in `readers` after the carry, and a row carrying an unread chase and an unread sender notice past retention, refolding the journal, and asserting that the refolded pool's `programs` holds no `<shortname>:CREEL` entry, the condition on which `checkReceiverLocked` refuses a post to `CREEL` as an unknown receiver, and that the carried messages stay with their holders.
`internal/creelengine/doc.go` gains a `# Reply discipline` section stating the chase, the clock and the outcome, and its Storage retention sentence, its Log section's events and its Delivery loop section name the chased message and the chase timer; `Scheduler.Stop`'s doc comment names the chase clock it persists; `docs/overview.md`'s creel row gains one sentence on it.

**ImpactSummary:** New chase timer and journal events inside creelengine; the fold, retention, read and wait-answer paths gain the chase state, and no existing message, inject or hold behavior changes for an unmarked post.
