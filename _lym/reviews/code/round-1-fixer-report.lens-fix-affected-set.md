```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1 (callers of `beamStart`):** The word-bounded search over `.go` and `.md` files found no remaining reference to the removed `beamStart` in `fabriccli`. The only hits are an unrelated local variable in `internal/shortcutcli/shortcut_test.go`. The `EnsureDobbies` seam was the only call site, and it now goes through `startDobbiesThroughBeamStart`.
- **F2 (sites choosing prime versus beam location):** The search for `ResolveWorktree(fabricengine.WorktreePath(` finds the `orchcli` site, now behind `bringUpLocation`. It also finds sites in `internal/beamcli` (`start.go`, `target.go`, `orchview.go`, `deps.go`, `cli.go`). These belong to `lym beam start`'s own resolution, not to `BeamUpWorktree`, so the fix did not need to cover them.
- **F3 (stale "alone" wording):** The search for "dobby alone", "starts the dobby alone" and "alone through" finds no other statement that a dobby starts alone. The remaining "alone" hits are unrelated. One of them is `internal/orchcli/cli.go:116`, "leaves the dobby alone on a bring-down", which concerns bring-down, not start.

Tool calls made:
1. One `grep` run covering the `beamStart` search, the "alone" search and the `ResolveWorktree(fabricengine.WorktreePath(` search. The changed files in my context cover only the files the review named, so a tree-wide search was needed.
