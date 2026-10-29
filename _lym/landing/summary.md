# Fix refused-forward chase subtest racing alpha's delivery worker

`TestScheduler_Chases/AChaseTheHolderStillHoldsUnreadIsRefusedForwardingAndStaysWithItsHolder` in `internal/creelengine/deliver_test.go` failed about two runs in three since the creel stage 3 change.
The production code is correct: the refused `Forward` writes nothing, because its refusal comes before any journal append.
The race was in the test.
Alpha's delivery worker, a real goroutine, typed the chase and called `store.markNoticeRead`, which appended a `read` journal entry and emptied alpha's unread inbox.
The test waited only for the chaser to re-arm its timer, so the worker's entry landed between the `before` journal read and the assertions.

This change touches only that subtest row.
The question is now posted by `strand(orch)` instead of `strand(beta)`.
After alpha reads it, the row holds alpha with `Store.Hold(strand(orch), alpha, time.Hour)` before the chase ticks.
Under that hold the worker types only orch's mail, so the chase from creel stays unread and the worker writes no journal entry for it.
The row still covers a chase its holder holds unread, and the refused forward is still checked against the journal length and alpha's unread senders.

## Design decisions

- **The hold is by the question's sender.**
  `ageChases` exempts only a hold by the message's own sender, so a hold by any other party stops the chase clock and the first chase never fires.
  `Hold` refuses a non-privileged strand holding another, so beta cannot hold alpha.
  Orch is the privileged strand sender the hold-clock row already uses, so the question moved to orch.
- **No `awaitChaseLines` call.**
  Under the hold the chase is never typed, so there is nothing for the worker to settle.
  The rearm wait in `chaseTick` already orders the chase's own journal records before the `before` read.
- **Test speed is unchanged.**
  The row keeps its injected clock and `t.Parallel`, and adds no real-time sleep.
  The one-hour hold is far longer than the clock advance of the row's chase ticks.

## To check

The row was not run here.
Run it under `lym gate test ./internal/creelengine` and confirm it passes on repeated runs, since the original failure was intermittent.
