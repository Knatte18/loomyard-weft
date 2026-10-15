# Round 3 fixer-report — plan

## Changes

- **F1** (fixed as proposed):
  card 07's `creelfake.Delivery` scripts `Idle` and `Paused` errors;
  the `Interrupt` and `Inject`-with-interrupt order refuses naming the error when `Delivery.Paused` answers one, so an unreadable pause record fails closed;
  `TestStore_InjectAndHold` gains a row for both forms under a scripted `Paused` error.
  Card 09's Intent states that an `Idle` or `Paused` error types nothing for that probe, is logged at Warn once per change of error, counts as a wake that delivers nothing and is retried;
  `TestScheduler_Injects` gains a scripted `Paused` row and a scripted `Idle` row asserting zero `Send` calls and the Warn line.
- **F2** (fixed as proposed):
  card 07 signals the wake notification on reply too, and `Reply` to a strand or program signals it as `Post` does;
  card 09 resets the probe interval on any wake notification (post, reply, forward, inject or release) and its `Scheduler` launches workers at start for every receiver the rebuilt pool shows pending;
  `TestScheduler_Reminders` gains a reply row and a row for the scheduler started over pending unread mail, so the start claim has a test.
- **F3** (fixed as proposed):
  `TestSweep_RemovedHolders`'s one-step return row gains an `orch-2` variant asserting one `withheld` line naming `creel-notices`, zero `Idle` calls and nothing typed.
- **F4** (fixed as proposed):
  card 07 states the default `creelfake` pane pid (a fixed pid that is neither 1 nor `os.Getpid()`);
  card 10's and card 14's EXTERNAL rows say they rely on it, and why.
- **F5** (fixed as proposed):
  card 07 states that `Inject` with `interrupt` queues at the head after `Delivery.Interrupt` returns nil or `ErrNoRunningTurn` (reported as nothing interrupted) and after any other error returns it with nothing queued;
  `TestStore_InjectAndHold` gains the `ErrNoRunningTurn` row and the other-error row.
- **F6** (fixed as proposed):
  card 09's What-counts text names the reset events, matching the Intent.

## Disputed

None.

## Deferred

None.
