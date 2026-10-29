# Card 4 — ring-claims-mail

**Edit:**
- `internal/creelengine#Request`
- `internal/creelengine#Response`
- `internal/creelengine#VerbHookWait`
- `internal/creelengine#Store.HookWait` -> `func (s *Store) HookWait(sender Sender, takesMail bool) (*Waiter, error)`
- `internal/creelengine#Waiter`
- `internal/creelengine#Waiter.Cancel`
- `internal/creelengine#Outcome`
- `internal/creelengine#OutcomeRing`
- `internal/creelengine#wakeMark`
- `internal/creelengine#Reading`
- `internal/creelengine#Reading.Ack`
- `internal/creelengine#Store.ClaimAtBoundary`
- `internal/creelengine#Store.ringHook` -> `func (s *Store) ringHook(strand Strand, ids []int64, now time.Time) bool`
- `internal/creelengine#worker.wakeHookStrand`
- `internal/creelengine#daemon.dispatch`
- `internal/creelengine#daemon.handOut`
- `internal/creelengine#daemon.awaitOutcome`
- `internal/creelengine_test#TestScheduler_HookWake`
- `internal/creelengine_test#TestDaemon_Protocol`
- `internal/creelengine/doc.go`

**Create:**
- `plan:internal/creelengine#Store.claimLocked` -> `func (s *Store) claimLocked(holder string, reading *Reading)`
- `plan:internal/creelengine#daemon.awaitAck` -> `func (d *daemon) awaitAck(conn net.Conn, reader *bufio.Reader, sender Sender, reading *Reading) error`

**Uses:**
- `plan:internal/creelengine#Store.FailJournalWrites`
- `internal/creelengine_test#env.start`
- `internal/creelengine_test#awaitOutcome`

**Intent:**
Decision D3, daemon side: the ring hands the mail over to a waiter that takes it.

Protocol: the struct `Request` gains `TakesMail bool` tagged `json:"takes_mail,omitempty"`, the hook-wait request's mark that its waiter takes mail; an older daemon ignores it (`PATTERN-tolerant-records`).
`daemon.dispatch` passes it to `Store.HookWait`, re-signed with `takesMail`, which records it on the new waiter in the struct `Waiter`'s new unexported field `takesMail bool` under the store's mutex.
The comments of `VerbHookWait` and of `Response.Messages` and `Response.AckRequired` say a ring to a marked waiter carries the claimed messages and asks their acknowledgement on the same connection, as a read and a claim do.

Claim: the budget loop of `Store.ClaimAtBoundary` moves into `Store.claimLocked`, which reserves to `reading` the holder's `claimableLocked` messages oldest first (hold owner's only under a live hold, none past `hookFieldCap`) and stops before the message whose line and joining newline would take the lines already in `reading` past `boundaryClaimBudget`, always taking the first; `ClaimAtBoundary` calls it unchanged in behaviour.

Ring: `Store.ringHook` takes the `Strand` so it can read the pause.
When a registered waiter of the strand takes mail, it reads `s.delivery.Paused(strand)` just before its locked step, as a boundary claim does, and with a pause or a pause read error it rings nothing and leaves the waiter registered.
In the locked step, with the claimable set still `ids` (a set that changed meanwhile rings nothing and leaves the waiter registered, as today), it claims through `claimLocked` into a new `Reading` whose new unexported field `ring bool` is set, and answers the waiter with `Outcome{Kind: OutcomeRing, Reading: reading}` (the struct `Outcome` gains `Reading *Reading`, nil for every other outcome) and records the wake mark of `ids` at now, unanswered.
`claimLocked` always takes the first message of a non-empty set, so once the set equals the non-empty `ids` the claim is never empty: D3's "a ring whose claim comes back empty does not ring" is met by the pause return and the changed-set return, the only states in which the ring's claim would come back empty, and no separate empty-claim branch is written.
An unmarked waiter, an older binary's, is rung as today with no `Reading`.
Either ring starts the strand's Stop count over.
`worker.wakeHookStrand` passes the strand.
`OutcomeRing`'s comment says a ring to a marked waiter carries the claimed mail and one to an unmarked waiter carries none.

Acknowledgement: `Reading.Ack` of a ring reading, once every id is journaled, marks the strand's last ring answered, so the claim answers the ring at the acknowledgement and not at hand-out; an Ack that fails leaves the ring unanswered (card 3-ack-failure-releases's release applies), so the bell's 30-second window follows it.
`wakeMark`'s doc comment says a ring is answered by the acknowledgement of its own claim, by a claim that handed out mail, or by a probe that reads a turn in progress after it.

Daemon: `daemon.awaitAck` holds the acknowledgement half of today's `daemon.handOut`: read the ack line under `ackReadTimeout`, release on any other answer or none, `Reading.Ack`, and write the confirmation or the refusal; `handOut` writes its response and calls it.
In `daemon.awaitOutcome`, an outcome carrying a `Reading` stops the close watch first, since the watch reads the connection the acknowledgement arrives on, then writes `Response{OK, ID, Sender, Outcome: ring, Messages: reading.Messages, AckRequired: true}` and hands the reading to `awaitAck` whether or not the write succeeded: on a connection the write failed on, `awaitAck`'s read of the ack line fails and releases the reading, so the ring path has one release step after hand-out, the one the client-leaves row reaches.
`Waiter.Cancel`, after unregistering and outside the store's mutex, drains an outcome still unread in the waiter's channel and releases its `Reading`, so every end of the ring's answer path before the outcome is taken, the client leaving or the daemon stopping while the outcome stands in the channel among them, releases the claim; `awaitOutcome`'s close and shutdown arms return without taking the outcome and run that `Cancel` from their defer, and an outcome already taken leaves the channel empty.

Tests (tier0), in `TestScheduler_HookWake`, with a new `registerTaking` closure beside `register` (which stays unmarked):
- `AMarkedWaiterIsRungWithTheBudgetedClaimAndItsAckJournalsItAndAnswersTheRing`: the turn reading is idle with a turn end a minute before the ring, two bigBody posts land before the scheduler starts, and the ring's `Reading` holds #1 only; after Ack the log holds #1 read; a second marked waiter registers after the Ack and stays silent, since the turn end precedes the ring, and 45 s of probes type no bell, so only the Ack's answered mark holds the 30-second bell back from the set {#2}; a later tool-boundary claim hands out #2.
- `AMarkedRingStartsTheStopCountOver`: on a fresh store, two bigBody posts and a marked ring of #1, acknowledged; a Stop claim with `StopHookActive: true` hands out #2, which a count no claim of this daemon set would refuse.
- `AMarkedWaiterUnderAnotherSendersHoldIsRungWithTheOwnersMailOnly`.
- `AMarkedWaiterIsRungWithoutALineOverTheHookFieldCap`: a `PostUnchecked` line over the cap and a short post; the `Reading` holds the short one.
- `APauseReadAtTheRingRingsNothingAndKeepsTheWaiter`: `SetIdleHook` sets alpha paused during the probe's Idle call, so locate passed and the ring's pause read finds it; the waiter is silent; after `SetIdleHook(nil)` and `SetPaused(alpha, false)`, a new post rings that same waiter.
- `APauseReadErrorAtTheRingRingsNothingAndKeepsTheWaiter`: as the row before, with `SetIdleHook` setting `SetPausedError` instead; after the hook and the error are cleared, a new post rings that same waiter.
- `AClaimTakingTheSetBeforeTheRingRingsNothingAndKeepsTheWaiter`: `SetIdleHook` runs a tool-boundary claim of alpha during the probe's Idle call, which reserves the set; the marked waiter is silent; after that claim's Release and `SetIdleHook(nil)`, a new post rings that same waiter.
- `ARingOutcomeNeverTakenIsReleasedWhenItsWaitIsCancelled`: after the ring the channel holds one outcome; `Cancel` without reading it; the bell follows at least 30 s after the ring, and then a claim hands the mail out.
- `AReleasedRingClaimLeavesTheMailUnreadAndTheBellFollows`: the taken `Reading` released; bell at ≥ 30 s; a claim hands the mail out.
- `AJournalFailureAtTheRingsAckLeavesTheRingUnanswered`: `FailJournalWrites(0)`, Ack fails, the bell follows the window.
- `AClaimableSetStandingBeforeTheWaiterRegistersRingsAtOnce` also asserts the unmarked ring's `Reading` is nil and a claim afterwards hands the mail out.
`register`'s and `OnlyAStrandRegistersAWaiter`'s `HookWait` calls pass false.
In `TestDaemon_Protocol`: `HookWaitIsPendingAtOnceAndAPostToTheIdleStrandRingsIt` also asserts the unmarked ring carries no messages and no `ack_required`; new rows `AMarkedHookWaitIsRungWithTheMailAndItsAckJournalsTheRead` (raw `takes_mail` request, ring response with one message and `ack_required`, ack, `ok` confirmation, log holds the read) and `AMarkedHookWaitWhoseClientLeavesBeforeTheAckLeavesTheMailUnread` (close after the ring; a strand `Request{Verb: VerbRead, Wait: true}`, which waits out the daemon's release as `ReadAckMarksReadAndAClientDyingBeforeTheAckLeavesMessagesUnread` does, hands the message out).

`internal/creelengine/doc.go`'s Idle wake section: the mark, the ring's claim under the boundary rules, the changed-set and paused rings that ring nothing, the acknowledgement answering the ring, every release path, an unmarked waiter rung as before, and the Bound that a pause starting between the pause read and the locked step lets one ring deliver; the sentence "A ring carries no mail and claims nothing" and the Bound "one ring costs one short model turn" go, and the claim paragraph's "delivered once, by whichever path reserves it first" names the ring's claim among the paths.

**ImpactSummary:** HookWait and ringHook re-signed with all callers on this card (dispatch, wakeHookStrand, two test sites); the read and claim acknowledgement paths keep their behaviour through awaitAck; all inside creelengine.
