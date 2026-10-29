# Hold the chase holder in the refused-forward chase row

`TestScheduler_Chases/AChaseTheHolderStillHoldsUnreadIsRefusedForwardingAndStaysWithItsHolder` in `internal/creelengine` failed about two runs in three.
After the chase ticks, alpha's delivery worker, a real goroutine, could type the chase and mark it read between the row's journal count and its asserts, adding a `read` entry and emptying alpha's unread inbox.

The row now posts its question from `orch` and holds alpha under `orch` before the chase ticks.
Under that hold the worker delivers only `orch`'s mail, so the chase from creel stays unread and the worker writes nothing the row asserts on.
The production code is unchanged: the refused `Forward` already writes nothing.

## Design decisions

- The hold belongs to the question's sender.
  A hold by anyone else stops that message's chase clock, so a hold from a program such as batten would never reach the first chase,
  and `Hold` refuses a plain strand such as beta holding another strand, so the question moves from beta to the privileged `orch` strand.
- No `awaitChaseLines` wait is added: under the hold the chase is never typed, and the chase tick's rearm wait already orders the chase's own journal records before the count.
- The row still covers a chase its holder holds unread; refusing the forward of an already-read chase would have dropped that case.
