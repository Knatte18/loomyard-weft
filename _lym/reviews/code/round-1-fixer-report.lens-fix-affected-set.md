```yaml
lens: fix-affected-set
outcome: reported
```

Re-ran the set-premised searches for F1, F2 and F4.

**Findings I left unchecked:** none. F3 concerns one missing test row, not a set.

**Findings**

- **LOW**: `internal/orchroute/live.go:69` and `:71`, the F4 change. Both rewritten calls are not gofmt-formatted: they read `downReadingLogMessage,"worktree"`, with no space after the comma.
  - Evidence: my grep output shows `logger.Debug(downReadingLogMessage,"worktree", ...` and `logger.Info(downReadingLogMessage,"worktree", ...`.
  - This is already committed in `31f832e21`. `lym loom fmt` was run before that edit and did not include `live.go`.

**Coverage**

- **F1:** every down-cause value outside tests now appears only in its constant declaration, including the new `pane-placement-unread`. The remaining literals are expected values in tests, plus an unrelated card name in `docs/research/.../forks.tsv`.
- **F2:** both callers of `strandsElsewhereLocked` (`lifecycle.go:1344`, `reconcile.go:217`) and the one caller of `downCause` match the new signatures. No other site lists the causes.
- **F4:** `downReadingLogMessage` is the only spelling of the log message. The other "reads as down" hits are prose comments and test names.

**Tool calls**

- grep for the down-cause values across the repo: my context held only the pre-fix search, limited to some symbols and file types.
- grep for "reads as down" and the new constant: needed to check the post-fix `live.go` and any twin literal.
- grep for the callers and the "found nowhere" / "down cause" wording: needed to check the post-fix signatures and stale prose.
