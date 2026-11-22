```yaml
lens: fix-proposal
outcome: reported
```

No findings.

Both proposals hold against the code:
- **F2.** The new summary text matches the seeding contract in `internal/configsync/configsync.go:415-427`. The walk calls `ReconcileHubWideAt` with `SeedDefaults: true` at `internal/hubreconcile/walk.go:225`, before `applyReinterpretations`. The new text claims no more than the docs give: an absent per-worktree default is seeded from the prime's copy, else as the header-only base.
- **F3.** The literal `"orch-remote-control-off-by-default"` equals `orchengine.RemoteControlOffReinterpretationID` (`internal/orchengine/reinterpret.go:12`), so the assertion is not vacuous. It would fail if the orch row lost its entry.

Calls made:
1. `grep`, `sed` on `internal/orchengine/reinterpret.go`, `internal/configsync/configsync.go` and `internal/hubreconcile/walk.go`: confirmed the ID constant's value and the seeding order. My context held neither.
