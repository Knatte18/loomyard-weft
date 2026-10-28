# Card 10 — The daemon rings an idle hook strand's waiter once per claimable set

**Create:**
- `plan:internal/creelengine#VerbHookWait` -> `const VerbHookWait = "hook-wait"`
- `plan:internal/creelengine#OutcomeRing` -> `const OutcomeRing OutcomeKind = "ring"`
- `plan:internal/creelengine#OutcomeSuperseded` -> `const OutcomeSuperseded OutcomeKind = "superseded"`
- `plan:internal/creelengine#waitHook` -> `const waitHook waitKind = 3`
- `plan:internal/creelengine#wakeMark` -> `type wakeMark struct`
- `plan:internal/creelengine#Store.HookWait` -> `func (s *Store) HookWait(sender Sender) (*Waiter, error)`
- `plan:internal/creelengine#Store.claimableLocked` -> `func (s *Store) claimableLocked(holder string) []*Message`
- `plan:internal/creelengine#Store.resetStopBlocks` -> `func (s *Store) resetStopBlocks(strand string)`
- `plan:internal/creelengine#worker.wakeHookStrand` -> `func (w *worker) wakeHookStrand(strand Strand, now time.Time) bool`
- `plan:internal/creelengine_test#TestScheduler_HookWake` -> `func TestScheduler_HookWake(t *testing.T)`

**Edit:**
- `internal/creelengine#Store`
- `internal/creelengine#OpenStore`
- `internal/creelengine#Store.ClaimAtBoundary`
- `internal/creelengine#worker.probe`
- `internal/creelengine#daemon.dispatch`
- `internal/creelengine#Client.resume`
- `internal/creelengine#Request`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestDaemon_Protocol`
- `internal/creelengine_test#TestDaemon_Lifecycle`

**Extends:**
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `internal/creelengine#waitKind`
- `internal/creelengine#Store.addWaiterLocked`
- `internal/creelengine#Store.answerLocked`
- `internal/creelengine#Store.wakeLocked`
- `internal/creelengine#Store.liveHoldLocked`
- `internal/creelengine#Store.unreadLocked`
- `internal/creelengine#daemon.awaitOutcome`
- `internal/creelengine#Waiter`
- `plan:internal/testkit/creelfake#HookStrand`
- `plan:internal/testkit/creelfake#Delivery.SetTurn`
- `internal/creelengine_test#newEnv`
- `internal/creelengine_test#newRig`

**Intent:**
D4: an `asyncRewake` waiter, started by every Stop of a strand's session, holds a waiting daemon call registered for its strand; the daemon wakes an idle strand by answering it, and the woken turn's first tool call (D1) or its end (D2) delivers the mail in a block the operator sees.
The ring carries no mail and claims nothing.

Waiters.
`Store.HookWait(sender)` registers a wait under `waitKey{kind: waitHook, holder: sender.Name}` and wakes the strand's worker, so a set already standing can ring at once; any sender but a strand is refused with "creel: only a strand's idle waiter hook waits for a wake, and <name> is no strand; read with lym creel read".
A newer waiter of the same strand ends the older: `HookWait` answers every wait already under that key with `OutcomeSuperseded` before it registers the new one.
A waiter counts as live while it is registered; `daemon.awaitOutcome` cancels it when its connection closes or the daemon stops, so the registration ends with the connection.
`daemon.dispatch` serves `VerbHookWait` by `HookWait` and `awaitOutcome` with id 0: a pending response at once, then `OutcomeRing` or `OutcomeSuperseded`.
`Request`'s `Wait` doc names hook-wait among the waiting verbs, and a hook-wait request carries `Wait: true` so its client reconnects;
`Client.resume` resumes a dropped hook-wait by sending the same request again, as it resumes a read, so a waiter reconnects after a daemon restart up to `MaxReconnects` and renews its registration.

The ring.
`claimableLocked(holder)` returns the unread messages `ClaimAtBoundary` would hand out ignoring the budget, oldest first: unreserved, the hold owner's alone under a hold, and none whose line passes `hookFieldCap`; `ClaimAtBoundary` selects through it, so the two never disagree.
`Store` gains `wakeMarks map[string]wakeMark`, kept in memory alone, and `wakeMark` holds the claimable ids the last ring was for and the clock time of that ring.
The ring time stays on the mark until the next ring replaces it, whatever the set does meanwhile: a set that changes after a ring, such as mail arriving in a woken turn's blocked Stop chain, changes the ids the next ring would be for but never clears the time the hold below rests on.
`worker.probe`, after the inject loop, calls `wakeHookStrand` for a strand whose `CreelHooks` is set, in place of the pane path, which stays for a pane-mail strand as card 9 left it; the pane lock and the pause skip `probe` already applies hold for it too, so under a pause nothing rings.
`wakeHookStrand` rings when the strand's claimable ids are non-empty and differ from its mark's, a live waiter is registered for it, and `Delivery.Idle` reads no turn: `NoTurn` true and, when the mark has a ring time and the answer a non-zero `TurnEndedAt`, that turn end after the ring.
That last rule holds the reading false from a ring until a turn end recorded after it, since a woken turn records no `UserPromptSubmit`, so mail arriving in a woken turn's blocked chain never finds the waiter a blocked Stop started and rings mid-chain; a reading the pane decided has a zero `TurnEndedAt` and is taken as it reads.
A draft does not hold the ring, since `NoTurn` ignores the box.
To ring it answers the strand's waiters with `OutcomeRing`, resets the strand's Stop count with `resetStopBlocks`, so a woken turn starts a fresh count, and records the mark; it reports a delivery, so the worker returns to its base wait.
The set changes when new claimable mail arrives or a claim takes some, so the next probe may ring again for the new set; an unchanged set is never rung twice.
Bound: one ring costs one short model turn before the mail turn; a ring that races a turn the operator starts delivers nothing and leaves one stale bell reminder; a restarted daemon has no marks and may ring a set once more.

Tests.
`TestScheduler_HookWake` drives a store over the fake delivery and clock with `HookStrand` strands, its rows:
a post to an idle hook strand with a live waiter, its box holding a draft (`SetTurn` running false, a draft stall), answers the waiter `OutcomeRing` and types nothing into the pane;
a second probe of the unchanged set rings no newly registered waiter, and a new post rings it;
a running turn (`SetTurn` running true) and no registered waiter each ring nothing;
a pause rings nothing; under another sender's hold only the owner's mail makes a set;
a newer `HookWait` answers the older `OutcomeSuperseded`;
after a ring, a new post with a turn end stamped before the ring rings nothing, and once the turn end is stamped after the ring the remaining set rings;
mid-chain: after a ring, a Stop claim takes part of the set, a newer `HookWait` registers as the blocked Stop's waiter would, and a new post arrives, changing the set, while the turn end stays stamped before the ring; nothing rings until a turn end stamped after the ring;
after a ring, a Stop claim with `StopHookActive` true hands out mail, the count `resetStopBlocks` reset, where the daemon-start cap would have refused it;
the cap chain: with 5,000-byte bodies so each claim takes one message, a Stop with `StopHookActive` false and five more with it true each hand out one, the seventh claims nothing while mail stands; a waiter registered after it, with a turn end stamped after that capped Stop, is rung for the remaining set; and the next Stop with `StopHookActive` true hands out mail;
a strand sender is required: `HookWait` from a program is refused naming `lym creel read`;
a `LiveStrand`, a pane-mail strand, is never rung.
`TestDaemon_Protocol` gains a row: with the rig's strand re-scripted as `HookStrand` with the test process's pane pid, since the rig's default `LiveStrand` is a pane-mail strand that never rings, a hook-wait over the socket gets its pending response, and a post to the strand rings it with `OutcomeRing`.
`TestDaemon_Lifecycle` gains a row on the same re-scripted `HookStrand`: a hook-wait whose daemon is stopped and started again reconnects, and a post to the new daemon rings it.
The doc gains the idle-wake section.
`pattern/PATTERN-daemon-wakeups.md`'s list of creel's waiting verbs gains `hook-wait`, which waits on its connection and reconnects from one second like the others.

**ImpactSummary:** a new daemon verb, a waiter registry and a ring step in the worker for marked strands, all inside creelengine; ClaimAtBoundary's selection moves into a shared helper with its behaviour unchanged.
