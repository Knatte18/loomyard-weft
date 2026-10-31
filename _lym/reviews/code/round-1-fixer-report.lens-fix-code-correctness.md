```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- **F2 row fails with the defect in place.** I traced `events: "START\nSTART:what are you doing?\nSTOP:answered\n"` with `session: true` and a `HoldDeferral` that always reports false.
  - Wait reads all three lines in one batch, and both starts precede the stop, so `turnStartRead` is false.
  - The first START is the prompt turn (`promptLine == ""`), and the second carries an operator instruction not in the sent texts, so `operatorStartedTurn` is true.
  - Today the `case ... operatorStartedTurn` at `wait.go:319` fires, so nothing is queued and nothing is notified.
  - With that case removed, control reaches `case !run.turnStartRead` at `wait.go:327`, which sets `pendingHold`. After the 5 minute `sleepJump`, `waited` is at least `holdNoticeSettle` (45s) and the predicate is false, so `settleDeferredHold` calls `notifyHeld`. `len(got.notices)` then becomes 1 and the row fails.
  - `timeout: 2 * time.Hour` leaves the run to end as `OutcomeTimeout`, matching the check. These fields are the same ones the existing deferred rows use.
- **F1 doc sentences match the code.** `positionedTurnStarts` (`wait.go:972-974`) skips a signal whose `Raw` is empty or absent from the data. Both `runTurnStarts` and `operatorInstructions` go through it (`operatorInstructions` calls `runTurnStarts`), so the sentence holds for both.
- **F3 is comment-only.** There is no code or signature change, so no caller is affected.

Calls made (each for something my context lacked):
1. Read `wait.go:300-350` and `hold.go:55-145`. The context had no body for the arm at `wait.go:319-329` or for `settleDeferredHold` and `operatorStartedTurn`.
2. Read `wait.go:955-990` and grepped the `holdNoticeSettle` and `holdNoticeMaxDeferral` constants. This confirmed the `Raw` skip claimed in the F1 comment and the settle and maximum durations the new row relies on.
