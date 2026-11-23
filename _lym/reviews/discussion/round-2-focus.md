---
round: 2
exclude_lenses: []
focus:
  - "D4 in `_lym/discussion/decision-record.md`: for each site that refuses an old config name (local or worktree file, hub base file, profile block, board `run_settings`, `lym config <module> --set`, recipe setting reference), does the one way forward it prints reach a fix at that site, given that `lym config reconcile` runs `configsync.ReconcileAll` over a worktree's own and local files only and `--layer default` runs `ReconcileHubWideAt` without reinterpretations, and that Seed-Child's `writeRunSettings` in `internal/battenshed/seamchild.go` already wraps a refusal in `lym board upsert`? Does a test reach each way forward?"
  - "D6 and D7 in `_lym/discussion/decision-record.md`: `ReconcileHubWideAt` merges the seed-only `profiles.yaml` through `yamlengine.FollowTemplate` (`internal/yamlengine/follow.go`) before D6's move runs. When an operator edited a shipped profile's old key and the follow also adds the template's new path, does the stated collision rule keep the operator's value, and is there an acceptance criterion that pins it?"
  - "D7 in `_lym/discussion/decision-record.md`: `ensure` in `internal/hubreconcile/walk.go` writes the build stamp whenever `walkHub` returns not skipped. Does D7 state the real retry point after a failed board read or write, and its behaviour when the hub-wide write was skipped?"
  - "D5 in `_lym/discussion/decision-record.md`: when the hub base `loom.yaml` is absent, `ReconcileHubWideAt` seeds it from the prime's worktree copy through `sparseSeed` in `internal/configsync/configsync.go`. Does D5 move that copy's old keys before the prune?"
---

Focus restates only sites and claims the decision record rests on.
No lens is excluded: round 1 raised MEDIUM or worse findings from three lenses, and the fourth lens, walkthrough, raised none but its area is the walk order D6 and D7 touch.
