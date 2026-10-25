```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run the set searches behind F2, F3 and F4 and check the fix covers every member.

No findings.

- F4: a search for "child's loom config" and "read from the child" across `internal`, `PATTERN.md`, `pattern` and `docs` returns nothing. No other comment names the old config source.
- F3: `liftPending` has one write site outside the fix, `owned.go:94`. It runs inside the per-run loop before `triggerFor` and the new clear, so the clear covers it.
- F2: `migrateDraftStatus` has one caller, `board.go:95`, in the single `boardCriticalSection`. The `doc.go:156` sentence, "the next write sets `draft` on every open task...", still holds with the migration moved before `fn`.

Tool calls:
1. `grep` for the old-config-source phrases over `internal`, `PATTERN.md`, `pattern` and `docs`: F4's set; my context held only `deps.go`.
2. `grep` for `liftPending` in non-test Go files: F3's site set.
3. `grep` for `migrateDraftStatus` in Go and Markdown files: F2's caller set.
4. `grep` for "migrat" and "draft" in `doc.go`, `PATTERN.md` and `docs`: docs that state the migration order.
