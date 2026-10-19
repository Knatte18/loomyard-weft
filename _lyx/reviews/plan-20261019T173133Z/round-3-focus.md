---
round: 3
focus:
  - "In `_lyx/plan/09-creel-deliver-marked-probe.md`, settle whether the `awaitMarkedProbe` protocol can wait forever or fail by timeout in any interleaving of a post's wake-driven probe with the caller's `mark` and `seen` reads, given that `Store.do` runs its wake hook in the poster's goroutine before `post` returns and `creelfake.Clock.Advance` fires only timers already registered."
  - "In the same card, settle whether the stated bound on the helper's total clock advance is consistent with the retried passes the card itself describes, and shorter than the shortest hold, TTL, back-off or stall freshness (`stallFreshness` in `internal/creelengine/pool.go`) each check runs under."
  - "In the same card, settle which helpers post before `e.start` (so `Start` creates the worker with no wake signal and its first probe snapshots every post), and whether the card gives any of them a reshaping that changes the number of pause reads or pane reads their own comments rely on."
---

Round 2 left card 9's `awaitMarkedProbe` protocol as the one design question.
The entries ask for the protocol's correctness and bound to be checked against the card as it stands now.
