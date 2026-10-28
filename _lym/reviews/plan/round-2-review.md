---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/plan/07-chase-clock.md
    summary: Card 7 keeps CREEL out of the known-mailbox set only in applyPosted, so a CREEL-sent message that retention carries makes CREEL a known receiver through applyCarried
    fix: >-
      In card 7's Intent, under Journal and fold, extend the sentence that makes applyPosted leave programs alone for a sender whose program name is ChaserProgram.
      State that applyCarried leaves programs alone for a carried posted snapshot whose From parses with agentname.ParseProgram to ChaserProgram, exactly as applyPosted does.
      In the same card's tests, add a TestJournal_Retention row that carries an unread chase and an unread sender notice past retention, reopens the fold, and asserts that a post to CREEL is still refused as an unknown receiver and that the carried messages stay with their holders.
      No target changes, so no validate-plan overlay is needed.
  - id: F2
    severity: LOW
    class: decision
    location: _lym/plan/07-chase-clock.md
    summary: Card 7 does not say whether a chaser's first tick charges, so the row that expects chase 1 after 15 minutes of running time can land at 15 or 16 ticks
    fix: >-
      In card 7's Intent, in the paragraph that describes chaser.run, state whether the chaser probes the holder once when it starts and counts that reading as the previous tick's, so the first tick one agedPersistInterval later charges, or whether the first tick only records a reading and charges nothing.
      Then make the Shared Decision chase-clock-cadence sentence "a step lands at the tick that completes its interval, at most one minute of running time after it fell due" and the TestScheduler_Chases rows "chase 1 after 15 minutes of running time" and "one interval of running time after the restart" agree with the chosen rule by naming how many ticks each takes.
      No target changes, so no validate-plan overlay is needed.
---

### [MEDIUM] Retention can make CREEL a known mailbox through applyCarried

**Location:** `_lym/plan/07-chase-clock.md`, Journal and fold bullets on `applyPosted` and `applyCarried`.

**Issue:** The focus asks whether any CREEL-sent `posted` record can make the CREEL program a receiver that posts are accepted for and nobody reads.
`checkReceiverLocked` (`internal/creelengine/pool.go`) accepts any name in `pool.programs`, and today both `applyPosted` and the `Of == eventPosted` branch of `applyCarried` (`internal/creelengine/fold.go`) set `programs[rec.From]` for a program sender.
Card 7 closes the first path ("`applyPosted` leaves `programs` alone for every `posted` record whose sender's program name … is `ChaserProgram`") and says nothing of the second.
`carryRecords` carries every unread message recorded before the retention cutoff, and a chase or a sender notice is an unread message from `CREEL` held by a strand that may not read for a week.
After such a carry, a restart folds the snapshot through `applyCarried`, `programs[<shortname>:CREEL]` becomes true, and `lym creel post <shortname>:CREEL "<text>"` is accepted into a mailbox nobody reads.
The other in-plan routes are closed: card 8 refuses a `Reply` to a CREEL notice, `Forward` and `post` go through `checkReceiverLocked`, and the `applyRead` route needs a CREEL read, which no verb makes.
The retention rows card 7 adds (a read, marked, unanswered message, and a program whose only read is past retention) do not reach a carried CREEL-sent snapshot, so nothing in the plan would catch it.

### [LOW] The first chaser tick has no previous reading

**Location:** `_lym/plan/07-chase-clock.md`, the clock paragraph on `chaser.run`, and the `TestScheduler_Chases` rows.

**Issue:** The card says a tick charges one interval only when its probe and the previous tick's both read the clock running.
It also says the chaser "ticks every `agedPersistInterval` from the moment it starts", with no probe at the start.
For the first tick there is no previous reading, so the plan leaves open whether it charges.
If it does not, a message read at t0 gets chase 1 on the sixteenth tick and not the fifteenth, and the restart row "one interval of running time after the restart" lands one tick late; the plan's "at most one minute of running time after it fell due" tolerates that, but the rows say "after 15 minutes", and a test that advances exactly 15 chase waits passes under one reading and fails under the other.
The implementer has to choose, and the choice changes what the rows assert.

## Rejected

- Chase timer cadence (focus 1): a fixed wait while the clock is stopped is not what the plan prescribes.
  The stopped clock doubles from `agedPersistInterval` to `chaseProbeCeiling` and returns to the base on a running tick or a wake, which is `PATTERN-daemon-wakeups`' Backoff rule for a wake that runs `reed status`; the running wait is a fixed minute, above the one-second floor.
  A backed-off tick charges nothing, because a tick charges only when it and the previous tick both read running one `agedPersistInterval` apart, and `chase-aged` is journaled at each tick and when the clock stops, the grid an inject's consumed TTL uses.
  Stopped time charged across a flip between two running ticks is bounded by one minute, which the Shared Decision states.
  The chase waits (one to eight minutes) do not collide with the delivery workers' waits (up to eight seconds) or the sweep's, and card 7 narrows `isSweepWait` accordingly.
  Not a defect.
- Reply-wait reconnect (focus 2): card 5 answers a `Reply` to a non-EXTERNAL sender's marked message to any registered wait, and `settledOutcomeLocked` answers a wait registered afterwards, which `Client.resume` does as `VerbWaitReply`, with the oldest reply whose `ReplyTo` is the id, whatever its status.
  A reply that landed during the gap is therefore handed over at reconnect, including when the old connection's waiter swallowed it, and a reply to a chase carries the chased id as `ReplyTo` (card 8) so it is found the same way.
  The unanswered case sits after the reply case, so a late reply wins over an unanswered record.
  A message can end replied with no outcome delivered only if the client stops reconnecting, which is the client's `MaxReconnects` bound.
  Not a defect.
- Import allowlist of `internal/reportengine` (focus 4): `ComputeReplies` imports `internal/creelengine` and `internal/agentname`; card 11 names `internal/reportengine#allowedImports` under Extends and its Intent adds exactly those two paths to the list in `seam_enforcement_test.go`.
  Both are used, so `AssertImportAllowlistNoStale` does not report either as stale, and `internal/creelengine` does not import `internal/reportengine`.
  Not a defect.
- Refusal and decision record (focus 5): the decision record's Added after Discussion entry for review F5 amends D10 and its Constraint so the chase-forward refusal names only `lym creel post --reply <chase id>` and the former-holder reply refusal is a fail-loud guard with no reaching test.
  Card 8 and the two Shared Decisions follow that text; `Store.Forward` refuses a message that is not unread (`internal/creelengine/verbs.go`), so the second way forward and a former-holder input are unreachable, and the forward refusal is reached by the holder forwarding the unread chase it holds.
  The refusal of a reply to a CREEL notice is the closure of the known-mailbox path above, names `lym creel post <strand>`, and has a test row.
  Not a defect.

## Focus departures

None.
