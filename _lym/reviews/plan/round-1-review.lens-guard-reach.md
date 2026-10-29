```yaml
lens: guard-reach
outcome: reported
```

guard-reach lens: whether each guard, refusal, branch and test row the cards add can be reached.

**List: each guard and whether it is reachable**
- Card 3, `Reading.Ack` journal failure releases the failing id and every later one: reachable through `FailJournalWrites(1)` after a three-message claim (verbs.go:471-486). The same failure in a read, a claim or a ring goes through the one `Ack`, so the guard does not recur unguarded anywhere else.
- Card 4, unmarked waiter rung with no `Reading`: reachable through `register` passing false (`AClaimableSetStanding…` and the protocol row `HookWaitIsPending…`).
- Card 4, pause read at the ring rings nothing: reachable. `SetIdleHook` runs inside `Idle` after `locate` has already read the pause (creelfake.go:205-213, deliver.go:269), so the ring's own pause read is the first to see the pause.
- Card 4, pause read error rings nothing: reachable the same way through `SetPausedError` inside the idle hook.
- Card 4, set changed before the locked step: reachable. The idle hook runs a claim with no lock held, and the reserved ids drop out of `claimableLocked` (pool.go:446).
- Card 4, the claim is never empty once the set equals the non-empty `ids`: holds. `claimLocked` always takes the first message, so no separate empty-claim branch is needed.
- Card 4, `Waiter.Cancel` drains an untaken outcome and releases it: reachable. The store row cancels without reading the channel. In the daemon, the close and shutdown arms of `awaitOutcome` return with the outcome still buffered (server.go:416-419), and the deferred `Cancel` runs (server.go:389).
- Card 4, `awaitAck` read failure releases the claim: reachable through the protocol row where the client closes after the ring. The branch where the ring response write fails then reaches `awaitAck`, has no row of its own, and ends in the same release.
- Card 4, a failed `Ack` leaves the ring unanswered and the bell follows: reachable. The released ids restore the set to `mark.ids`, so `sameSet` holds, the mark is unanswered, and the window branch types the bell (deliver.go:263, 285-287).
- Card 4, Ack's answered mark holds the bell back: reachable. With a second waiter registered, the unanswered branch is tested before the `waiting` branch (deliver.go:285-289), so only the answered mark keeps the bell back.
- Card 4, `AMarkedRingStartsTheStopCountOver`: reachable. On a fresh store the count is unknown and refuses (verbs.go:329), and only the ring's reset admits the claim.
- Card 5, `ErrAckUnsent`: reachable. The fake closes before the handler returns, and a write to a unix socket whose peer has closed fails with EPIPE.
- Card 5, no resume once the handler ran: reachable through the lost-confirmation row (`awaitAckConfirmation` returns `connLostError`, client.go:278-279).
- Card 6, `hookWaitExit` branches:
  - Confirmation refused: reachable; the daemon's `!OK` reply becomes a plain error (client.go:285-286).
  - Lines not written: reachable through `failingWriter`.
  - Ack unsent: reachable through the card 5 branch.
  - Ring with no lines: reachable only against an older daemon, as D3 intends.
  - Superseded: reachable.
- Card 7:
  - Echo handed out under a pause or a hold: reachable.
  - Echo under the Stop cap: reachable on the seventh Stop claim.
  - Shared budget: reachable.
  - Echo of the ids before a failing ring Ack: reachable.
  - Claim Ack drops the echo despite a journal failure: reachable.
  - Echo of an id forwarded away before the Ack: reachable, since `Forward` does not refuse a reserved message (verbs.go:586-599).
  - Flag standing on an owed echo: reachable.
- Card 7, echo handed out while an unknown Stop count keeps the mail back: this combination cannot occur. An echo exists only after a ring of this daemon, which sets the count (verbs.go:451-453), and a restart drops the echo. No row claims it, so this is not a finding.
- Card 8:
  - Stop with an echo and no mail exits 1: reachable. After the ring the count is known and zero, and the CLI fake leaves `TurnEndedAt` zero, so the second ring also fires (creelfake.go:225-230).
  - Merged command records the turn end with the echo output: reachable through the stub row.
- Card 9, every `outcome` and `reason` row and every bell outcome row: reachable.
  - `stop-count-unknown` on a fresh store.
  - `held` through the Hold verb.
  - `not-landed`, `busy` and `failed` through `SetSendError`.
  - Ring `paused` through the idle hook.

**Findings**

1. LOW, design. Card 4 (`04-ring-claims-mail.md`, the Ring paragraph).
   `ringHook` reads the pause only when the registered waiter takes mail, decided before the locked step. The locked step then claims for whichever waiter is registered at that moment.
   `HookWait` supersedes inside the lock (verbs.go:407-415). So a marked waiter that replaces an unmarked one between the check and the locked step is rung with a claim that read no pause.
   The pause guard is therefore missing on that path. It needs mixed binaries, which only happen during a deploy.
   Evidence: verbs.go:395-421, 435-455.
   Fix: the locked step claims only when the pause was read for a marked waiter, and otherwise rings nothing and keeps the waiter.

2. LOW, consistency. Card 9 (`09-claim-delivery-log.md`, the bullet on `released`).
   `Reading.Release` logs `released` with no condition. `daemon.handOut` calls `Release` when the response write fails even for an empty claim reading (server.go:455-458).
   A claim that handed out nothing therefore logs a Debug `nothing` line and then an Info `released` line with empty ids: two lines for one attempt, against D5's "one line".
   Fix: `Release` logs only when the reading held ids or echo items.

3. LOW, consistency. Card 7 (`07-ring-echo.md`, row `ThePausedOrHeldStrandsClaimStillHandsOutTheEcho`).
   The row expects "the same" echo from the held claim after the paused claim. Card 7 has a claim's `Ack` drop its echo items (Intent, Claim paragraph).
   Unless the paused claim is released, or a second ring is acked before the hold half, the held claim's `Echo` is empty. The row does not say which, so its second assertion's state may not occur as written.
   Fix: release the paused claim, or split the row so each half starts from its own acked ring.

Tool calls:
- Read `internal/testkit/creelfake/creelfake.go` lines 195-272 (`SetIdleHook`, `Idle`, `SetTurn`, `Paused`): needed to confirm when the idle hook runs relative to the pause reads and what `TurnEndedAt` the fake gives. The manifest listed only the method names.
- Grep in `internal/creelengine` for `markRingAnswered`, `markBell`, `lastHookWaiterEnd` and the probe call site: needed to trace the bell and answer-mark states the card 4 rows rely on. These functions were outside the pre-read.
