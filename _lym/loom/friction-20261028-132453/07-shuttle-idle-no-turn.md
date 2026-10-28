# Card 7: the test engine already implements InterruptGuard

Card 7 says the unknown-reading and no-run-record rows run on a `readinessEngine` that implements no `InterruptGuard`, and creates `guardedReadinessEngine` to add one.
`fakeEngine` already implements `InterruptGuard` (`RunningTurn` on the needle `RUNNING`), and `readinessEngine` embeds it, so every row's engine is guarded.
Those rows therefore want `NoTurn` true for a busy pane holding a draft, not false as the card says, and `guardedReadinessEngine` pins no behavior the base engine does not.

A card-8 change (`PaneMail` on every persisted strand) broke a tier1 `creelcli` test the card gate (tier0 only) did not run; the batch gate found it.
