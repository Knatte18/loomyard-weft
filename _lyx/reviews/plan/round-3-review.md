---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/09-creelengine-delivery.md:19
    summary: >-
      An Idle or Paused error from the Delivery seam has no handling in the worker or in Store.Interrupt, and card 13's claim that the worker types nothing then has no test that reaches it
    fix: >-
      In card 07 (`_lyx/plan/07-creelengine-pool-verbs.md`) add scripted `Idle` and `Paused` errors to the `creelfake.Delivery` description (line 44), and state in the `Interrupt` and `Inject`-with-interrupt order (line 36) that a `Paused` error refuses with the error named and interrupts, queues and types nothing; add one such row to `TestStore_InjectAndHold`.
      In card 09 (`_lyx/plan/09-creelengine-delivery.md`) add to the Intent, after the pause sentence on line 19, that an `Idle` or `Paused` error types nothing for that probe, is logged at Warn once per change of error, counts as a wake that delivers nothing and is retried at the next probe; add one row for it to `TestScheduler_Injects` (already non-parallel because of `logcapture`) with a scripted `Paused` error and a scripted `Idle` error, asserting zero `Send` calls and the Warn lines.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-creelengine-pool-verbs.md:24
    summary: >-
      The per-receiver wake notification is signalled on post, forward, inject and release but not on reply, so a reply to a receiver with no worker starts none and is never reminded; card 09 resets the probe interval on post, inject and release only
    fix: >-
      In card 07 line 24 add `Reply` to the verbs that signal the receiver's wake notification (post, reply, forward, inject and release), and add to the `Reply` description on line 28 that a reply to a strand or program signals it as `Post` does.
      Add one `TestScheduler_Reminders` row to card 09 (line 34) where a reply to a live idle strand with no running worker is reminded once.
      In card 09 line 17 reset the probe interval on any wake notification (a new post, reply, forward, inject or release) instead of listing post, inject and release alone, and state on the Scheduler that at start it launches a worker for every receiver the rebuilt pool shows with a queued inject or unread mail not muted, so delivery after a daemon restart does not wait for a new post.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/09-creelengine-delivery.md:36
    summary: >-
      The sweep's return of mail to an orch strand is stated to journal a withheld event (line 20) but no TestSweep_RemovedHolders row reaches it
    fix: >-
      Extend the `TestSweep_RemovedHolders` row for mail returned one step so that one variant returns the mail to an `orch-2` strand, asserting one `withheld` line naming `creel-notices`, zero `Idle` calls for it and nothing typed. This is an assertion in an existing row, no new test function.
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/10-creelengine-daemon.md:38
    summary: >-
      EXTERNAL rows need live receiver strands that carry a non-zero pane pid outside the test process ancestry, or card 08 refuses the EXTERNAL peer on Linux, and no card says what pane pid those creelfake strands carry
    fix: >-
      In card 07 line 44 state that a `creelfake` strand scripted without a pane pid carries a fixed pid no ancestor of the test process holds (neither 1 nor `os.Getpid()`), and in card 10 line 38 and card 14 line 40 say that EXTERNAL rows rely on that default for their receiver strands, since a live strand with pane pid 0 makes card 08 refuse a peer in no known pane and a pid in the test's ancestry makes the peer that strand.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/plan/07-creelengine-pool-verbs.md:36
    summary: >-
      The outcome of inject with interrupt when Delivery.Interrupt answers ErrNoRunningTurn or another error is left open: whether the text is still queued at the head
    fix: >-
      In card 07 line 36 state the choice, which D10 supports as queue regardless: `Inject` with `interrupt` queues its text at the head after `Delivery.Interrupt` returns nil or `ErrNoRunningTurn` (reported as nothing interrupted), and after any other error returns that error with nothing queued. Add a row for the `ErrNoRunningTurn` case to `TestStore_InjectAndHold`.
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/plan/09-creelengine-delivery.md:32
    summary: >-
      The What-counts entry for the delivery workers states the doubling but not the reset events that the Intent states, and the TestScheduler_Reminders waits row names only two of them
    fix: >-
      In card 09 line 32 extend the What-counts text to read from 1 second, doubling to 8 seconds after every wake that delivers nothing and returning to 1 second on a new post, reply, forward, inject or release and on a wake that delivers, so the entry, the Intent on line 17 and the waits row on line 34 state one rule.
---

### [MEDIUM] Idle and Paused errors have no handling and no test

**Location:** `_lyx/plan/09-creelengine-delivery.md:19`, `_lyx/plan/07-creelengine-pool-verbs.md:36`, `_lyx/plan/13-creelcli-delivery-adapter.md:23`
**Issue:** Card 13 makes `Paused` return an error for an undecodable pause record and says creelengine's worker then types nothing for that probe.
That behaviour belongs to card 09's worker, and card 09 states it nowhere: its only error rule is for `Send` (line 21).
Card 07's `Interrupt` order refuses while `Delivery.Paused` covers the receiver but does not say what a `Paused` error does, which is the question of whether a failed pause read lets an interrupt through.
The `creelfake.Delivery` description (card 07, line 44) scripts send, interrupt and strand-list errors only, so no test can reach either path.
A guard branch with no named input and no named test is what the rubric's reachability item flags; this one guards the pause (D14), the safety check that must fail closed.

### [MEDIUM] Reply does not wake the scheduler

**Location:** `_lyx/plan/07-creelengine-pool-verbs.md:24`, `_lyx/plan/07-creelengine-pool-verbs.md:28`, `_lyx/plan/09-creelengine-delivery.md:17`
**Issue:** Card 09 starts or wakes a worker only through card 07's per-receiver wake notification, which card 07 signals on post, forward, inject and release.
`Reply` (D12: to a strand or program it is mail as any other) is not on that list and its description does not say it signals.
A reply to a live idle strand whose worker does not exist would sit unread and unreminded.
Card 09's reset rule also names only post, inject and release, so a forward the notification does signal does not reset the probe interval.
Nothing in card 09 starts workers for the pool rebuilt at daemon start either, so mail pending across a restart waits for the next notification.
No `TestScheduler_Reminders` row reaches the reply path.

### [LOW] Sweep return to an orch has no test row

**Location:** `_lyx/plan/09-creelengine-delivery.md:36`
**Issue:** Line 20 assigns the `withheld` event for a return to an orch to the sweep (a post, reply and forward are card 07's, and the return is the sweep's), and the answer to the orch-unread-set question is otherwise consistent: card 07 covers post, forward and reply, card 09 the return.
`TestSweep_RemovedHolders` has no row where the one-step-back target is an orch, so the sweep's write of that event is unreached.

### [LOW] EXTERNAL rows leave receiver pane pids unspecified

**Location:** `_lyx/plan/10-creelengine-daemon.md:38`, `_lyx/plan/14-creelcli-verbs.md:40`, `_lyx/plan/08-creelengine-identity.md:22`
**Issue:** Card 08 refuses a peer in no known strand pane when a live strand reports pane pid 0, and resolves the peer to a strand when a pane pid is in its ancestry.
Card 10 says an EXTERNAL row scripts no strand with the test's pid, and card 14's EXTERNAL rows post, inject and wait on receivers, which must be live `creelfake` strands.
Neither card nor card 07's kit says what pane pid those receivers carry, so an implementer who leaves the field at its zero value gets a refusal, and one who picks pid 1 turns the test process into that strand.

### [LOW] Inject with interrupt: outcome on a failed interrupt is open

**Location:** `_lyx/plan/07-creelengine-pool-verbs.md:36`
**Issue:** D10 says `inject --interrupt` sends the guarded interrupt, then queues its text.
Card 07 reports `ErrNoRunningTurn` as nothing sent for the `interrupt` verb and says `Inject` with `interrupt` then queues, but does not say whether the queue step runs when the interrupt found no running turn or failed for another reason, such as a dormant receiver.
The test rows cover the `interrupt` verb's `ErrNoRunningTurn` answer and not the inject form's.

### [NIT] What-counts entry omits the reset events

**Location:** `_lyx/plan/09-creelengine-delivery.md:32`
**Issue:** The doubling rule is the same in the Intent, the What-counts text and the `TestScheduler_Reminders` waits row (1, 2, 4, 8 and 8 seconds, to a named 8-second ceiling), including for dormant, busy and in-backoff receivers.
The reset events differ: the Intent names a new post, inject, release and a wake that delivers, the waits row names a new post and a wake that delivers, and the What-counts text names none.

## Rejected

- Card 09 delivery worker wake interval (focus 1): the worker doubles from 1 second to a named 8-second ceiling for busy, held, paused, dormant and in-backoff receivers, and resets on stated events; the departure from D19's list is justified in the card and `PATTERN-daemon-wakeups` rule wins, so the interval itself is not a defect; only the reset wording is F6 and F2.
- Cards 7 and 9 ownership of the interrupt and inject refusals (focus 2): card 07 lists `verbs.go` under Create and owns the privilege, orch, pause and hold checks in a stated order; card 09 only types the queued head and asks no later card to patch a file it does not list. Not a defect.
- Card 07 privileged-inject Bound (focus 4): it states that the TTL bounds staleness only while the receiver is live and unpaused, that a dormant strand's inject waits untyped and expires only when the sweep finds the strand gone, what the interrupt forms can do, and which rules limit them. Not a defect.
- Card 14 environment handling in `TestCreelCLI` (focus 5): the card states `t.Setenv` for strand rows and an empty `t.Setenv` for EXTERNAL rows, no `t.Parallel`, a comment naming the environment, and the platform where no peer is reported. Not a defect; the pane pid default is F4.
- Test files over `creelfake` (focus 6): `verbs_test.go`, `deliver_test.go` and `server_test.go` are `package creelengine_test`, while `journal_test.go`, `identity_test.go` and `testmain_test.go` stay in `package creelengine` and use no kit that imports `creelengine`; `creelfake.Clock` records the durations passed to `After` that cards 09 and 10 read. `internal/creelcli`'s in-package tests importing `creelfake` close no cycle. Not a defect.
