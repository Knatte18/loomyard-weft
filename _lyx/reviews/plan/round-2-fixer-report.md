# Burler plan round 2 — fixer report

## Changes

- **F1** (fixed as proposed): card 9's worker sentence now backs off after every wake that delivers nothing (busy, held, paused, dormant, or idle inside the reminder backoff) and resets on a new post, inject or release or a delivering wake, with the departure from D19 stated as `PATTERN-daemon-wakeups` winning.
  The What-counts text for the delivery workers and the `TestScheduler_Reminders` probe-interval row match it, the row now covering the dormant and in-backoff receivers and the reset after a delivering wake.
- **F2** (fixed as proposed): card 7's `Interrupt`/`Inject` sentences now own the whole chain in order (privilege, orch, pause, hold naming owner and expiry with the owner passing, then `Delivery.Interrupt` with `ErrNoRunningTurn` reported as nothing sent, then head-queueing), and the undefined "interrupt key" is gone.
  Card 9's two-sentence paragraph is deleted; its existing sentence "`inject --interrupt`'s queued head is typed at the next idle after `Interrupt` ran" keeps the head-typing.
  The pause, hold, owner-pass and `ErrNoRunningTurn` rows moved from `TestScheduler_Injects` to `TestStore_InjectAndHold`; card 7's Intent tag reads "D10 checks".
- **F3** (fixed as proposed): card 7's `Reply` journals one `withheld` event for an orch receiver, card 9's two enumerations name the reply, and `TestStore_Mail`'s reply row gains a reply to an orch message shown by log as one withheld line.
- **F4** (fixed as proposed): card 7's first Bound now covers the interrupt forms, limits the TTL to a live unpaused receiver, describes the dormant-strand case and its sweep expiry, and lists what holds typing back (pause, hold and orch rules of cards 7 and 9, card 5's pre-write pane guard, the non-privileged refusal).
  Card 16 does not restate the bound, so it is unchanged.
- **F5** (fixed as proposed): card 14's `TestCreelCLI` paragraph sets `LYX_STRAND_NAME` with `t.Setenv` in strand rows and to empty in every EXTERNAL row, and states that the test does not call `t.Parallel` and names the environment as process-global state.
- **F6** (fixed as proposed): card 7's `creelfake.Clock` records every duration passed to `After`, in order.
- **F7** (fixed as proposed): cards 7, 9 and 10 state that `verbs_test.go`, `deliver_test.go` and `server_test.go` are `package creelengine_test` files; card 7 also says `journal_test.go`, `identity_test.go` and `testmain_test.go` stay in package `creelengine`.

No target was added, dropped or moved, and no verify command changed.

## Disputed

None.

## Deferred

None.
