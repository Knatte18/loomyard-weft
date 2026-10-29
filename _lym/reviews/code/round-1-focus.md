---
round: 1
focus:
  - "internal/creelengine/deliver_test.go, the refused-forward row: settle whether a hold by orch on alpha leaves alpha's delivery worker unable to type or read the chase from creel, so no `read` journal entry can land between the `before` read and the asserts."
  - "internal/creelengine/chase.go `ageChases` and the row's `chaseTicks(15)`: settle whether the chase clock of the orch-sent question still reaches its first chase under orch's own hold, so `creelPostTo(e, creel, alpha)` finds a chase."
  - "The `Forward` refusal assert in the row: settle whether the expected `which answers #1` still names the question after posting it from orch instead of beta, and whether `Store.Hold(strand(orch), alpha, ...)` is permitted for that sender."
  - "Scope against the decision record: settle whether the diff touches only that one row and keeps the injected clock, `t.Parallel` and the unread-chase coverage, with no real-time sleep added."
---

The diff is one test row; the board entry's hold-from-another-sender suggestion was replaced by a hold from the question's own sender, so the clock and hold-permission questions carry the risk.
