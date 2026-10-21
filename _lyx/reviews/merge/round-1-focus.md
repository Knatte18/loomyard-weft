---
round: 1
focus:
  - "internal/hubreconcile/walk.go `reconcileHubWide`: the task added the base-sparse strip, the marker and `boardErrorFor`, and the parent replaced the in-walk push and `HasUpstream` branch with `spawnBoardSync`. Settle whether the merged function still compiles into one coherent flow: no leftover `pushErr`/`HasUpstream` use, no `boardErrorFor` caller on a path the parent removed, and the marker commit still reaches the board's remote through the detached sync."
  - "The stamp gate in `internal/hubreconcile/walk.go`: the task's `repoDefaultAbsent` treats an absent base-sparse marker as stale, and the parent's walk no longer reports skipped on a push failure. Settle whether a board whose strip commit is never pushed leaves a stamp that wrongly suppresses or wrongly repeats the strip, against `internal/hubreconcile/doc.go` as merged."
  - "`WorktreeError.Board` and the board way-forward text the task added, against the parent's reworded `reconcileFailureWayForward` and failed-push records in `internal/fabriccli/fabric.go`: settle whether the two way-forward wordings contradict each other, e.g. a `git pull --rebase` instruction the parent withdrew."
  - "Callers of `hubreconcile.Ensure` versus `EnsureOrWarn` in `internal/battencli/arm.go`, `internal/orchcli/cli.go`, `internal/loomcli` and `internal/preflightshed`: the task changed some sites and the parent added `reconcileResume` and `reconcile` hooks beside them. Settle whether a start verb still behaves as the merged docs and the loomcli smoke test say when the reconcile fails, including the order of the origin reconcile and the config reconcile."
  - "`PATTERN.md` and `pattern/PATTERN-recipe-setting-lookup.md`, both changed on each side: settle whether the merged text keeps the parent's new invariants and the task's sparse-base invariant without one overwriting or contradicting the other, and whether the task's code follows the parent's changed entries."
  - "Test economy in `internal/hubreconcile/ensure_integration_test.go` and `export_test.go`: settle whether tests the task added for push-failure or stamp-absent behavior still exercise code the parent removed, duplicate a parent test, or leave the strip-plus-detached-sync interaction uncovered."
---

Seed focus drawn from the files both sides touched; the walk, the stamp gate and the failure wording are where the two edits meet.
