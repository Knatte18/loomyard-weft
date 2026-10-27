---
round: 3
focus:
  - "Card `_lym/plan/08-dobby-worktree-gone.md`, row `TestRun_WorktreeGoneExits` for a worktree removed after tick 1 by `Deps.Sleep`: settle whether the row's assertions fail when the top-of-iteration worktree-gone check in `internal/dobbyengine/loop.go` is deleted, by naming the observable that differs, such as the tick count or a recreated `.lym/beam` stub at the removed path, and whether the harness `stopAt` leaves room for a second tick."
  - "Card `_lym/plan/08-dobby-worktree-gone.md`, the paragraph on the after-tick state read: settle whether every branch it prescribes, including a read error with the marker still present and a read error with the marker gone, has a named input and a named `TestRun_WorktreeGoneExits` row that reaches it, and whether the paragraph's error handling agrees with how `loop.go` treats a tick error."
  - "Card `_lym/plan/11-gate-adopt-key.md`, the Intent's sentence on a hub `gate.yaml` seeded from a prime copy or legacy files: settle whether it states a bound or carries the case, given that `hubreconcile.Ensure` returns after one build-stamp read once the stamp names the running build."
---

Round 3 re-checks the places the round 2 fixes touched.
