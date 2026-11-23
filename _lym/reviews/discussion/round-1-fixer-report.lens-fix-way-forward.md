```yaml
lens: fix-way-forward
outcome: reported
```

Checked every way forward, refusal and retry the fix introduced, including that each can be carried out from the state it names and leads to progress.

no findings

Each way forward checks out:
- **Local and worktree file:** `lym config reconcile --apply` in that worktree runs `configsync.ReconcileAll`, which D5 now makes move the keys.
- **Hub base file:** `lym config reconcile --layer default --apply` runs `ReconcileHubWideAt`, which D5 and D6 make move the keys.
- **Task `run_settings` at Seed-Child:** the `lym board upsert` way forward works. Upsert validates the new name through `PrepareRunSettings` and replaces the stored `run_settings` object whole, so no old key survives (`internal/boardengine/replay.go:208`).
- **Profile block:** the profile check refuses a write that holds an old key, so the operator writes the new name. Separately, D6's move now also runs on a default-layer reconcile.
- **D7 retry:** keeping the stamp stale reuses the existing path that a skipped hub-wide write already takes (`internal/hubreconcile/walk.go:81-84`), so the next start verb's walk reaches the retry.

Tool calls:
- Read `internal/boardcli/runsettings.go` (lines 120-160) to see how upsert validates `run_settings`. My context had only the call site in `seamchild.go`.
- Grepped `validateRunSettings` and the `RunSettings` setters to learn whether upsert merges or replaces `run_settings`. My context did not hold the upsert write path.
