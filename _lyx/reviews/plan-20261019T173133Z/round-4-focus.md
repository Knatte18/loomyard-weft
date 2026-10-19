---
round: 4
focus:
  - "Card 9 (`_lyx/plan/09-creel-deliver-marked-probe.md`), subtests that follow `awaitMarkedProbe` or `awaitTyped` with a further step that flips a fake or jumps the clock (`BusyReceiverIsRemindedAtItsNextIdleEdge`, `PauseTypesNothingUntilLifted`, `HoldMutesOtherSendersLetsTheOwnerPassAndEndsAtExpiry` in `internal/creelengine/deliver_test.go`): does the Intent state what the worker is doing when each helper returns, and does it leave no bare `tick(d)` or `probeWaits(before + n)` that a probe still in flight can race?"
  - "Card 9, subtests that call `holdAfterAReminderProbeReadTheDraft` and then `InjectWait` after `e.start` (`StallClearedUnderAHoldIsNotSeededIntoALaterWaiter`, `PersistingStallIsToldOnceToALaterWaiterAndEachChangeOnce`, `ExpiryNamesNoStallLastReadBeforeTheBound`): does the Intent name which wait rule governs them, given a `note.At.Equal(e.clock.Now())` assertion and a 16 s stall freshness that the helper's clock advance would reach?"
  - "Card 9, the stated bound on helper passes and the sentence on stall freshness: does the bound hold for a subtest that posts twice before its first helper call, and is the comparison with `stallFreshness` stated as the code in `pool.go` makes it?"
---

Focus names where the plan's last changes sit; it caps no severity and pre-states no verdict.
