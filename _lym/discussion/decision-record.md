# Decision record — chases-refused-forward-journal

## Goal

Make `TestScheduler_Chases/AChaseTheHolderStillHoldsUnreadIsRefusedForwardingAndStaysWithItsHolder` deterministic: alpha's delivery worker must never read the chase while the test asserts on the journal and the inbox.

## Scope

- In: that one subtest row in `internal/creelengine/deliver_test.go`.
- Out: production code in `internal/creelengine`, which is correct; the other chase rows.

## Decisions

### D1 — Hold alpha under the question's sender before the chase ticks

The question is posted by `strand(orch)` instead of `strand(beta)`, and after alpha reads it the test holds alpha with `Store.Hold(strand(orch), alpha, time.Hour)` before the chase ticks.
Rationale: a hold by any party other than the message's sender stops its chase clock, so a hold from `program(batten)` or `strand(alpha)` would never reach the first chase;
`Hold` refuses a non-privileged strand holding another, so beta cannot hold alpha, and orch is the privileged strand sender the hold-clock row already uses.
Under orch's hold the delivery worker types only orch's mail, so the chase from creel stays unread and the worker writes no journal entry for it.

### D2 — No wait on the worker

The row adds no `awaitChaseLines` call: under the hold the chase is never typed, so there is nothing to wait for, and `chaseTick`'s rearm wait already orders the chase's own journal records before the `before` read.
Rationale: the hold removes the worker's only effect on the asserted state, so the race has no window left.

## Constraints

- `PATTERN-test-speed`: the row keeps its injected clock and `t.Parallel`; no real-time sleep is added.
- The row still covers a chase its holder holds unread.

## Auto-mode assumptions

- A darn run holds no interview; the adjustment of the entry's suggested hold to the sender's own hold is settled from the code.

## Open risks

- None known: the hold lasts an hour and the row's chase ticks advance the clock by well under that.

## Acceptance criteria

- D1: the row posts from orch and holds alpha under orch before the first chase tick, and the forward of the chase is still refused with the way forward.
- D1, D2: the journal length is unchanged by the refused forward and alpha's inbox still lists the chase unread from creel, on every run.
- The row passes under `lym gate test` scoped to `internal/creelengine`.
