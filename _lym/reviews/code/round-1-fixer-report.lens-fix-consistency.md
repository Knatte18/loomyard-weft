```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Lens fix-consistency: I checked whether the changed text still agrees with its siblings and twins.

- **`TurnHookCommand` doc (`spec.go:152`).** The new wording matches `creelStopHookCommand`, which runs the hook step first (`settings.go:134-137`). `settings.go:429` and `:147` do not claim the hook skips turns where mail is handed out. No stale twin remains in the Go or markdown files I grepped, apart from the `_lym` tree and `_test.go` files, which I did not search.
- **`forward` signature.** `handle` at `inbound.go:657` is the only caller. The `handle` doc, the `awayFrom` and `awayPending` field docs, and the `takeAwayFrom` doc agree with the new post-based condition.
- **`noticePresenceChange` doc (`relay.go:680`).** It still describes the skipped set accurately.
- **Tests.**
  - The `TakeAwayFrom` wrapper in `export_test.go` follows the file's naming pattern.
  - The new outbound rows use `onReadWait`, `Floor` and `Ceiling` the way the existing rows do.

Calls made:
1. `grep` for the "no creel mail" doc phrasing, `TurnHookCommand` and `forward(` across the repo's `.go` and `.md` files, to find twins of the changed docs and any other `forward` caller. My context held only the files the fixer read.
