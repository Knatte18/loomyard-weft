```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: check whether the change set still agrees with its surrounding docs, patterns and twin sites.

**No findings.**

- **F1:** The new text in `pattern/PATTERN-run-fsm.md` ("names the owed record's own way forward, as `FailedPushWayForward` gives it") agrees with the restored `act.go` refusal switch and the restored `runread.go` caller. The `Aside.PushOwed` comment now lists the way forward, and `Judge` clears it, so the two match. The `doc.go` line that has `PushFailedError` append `FailedPushWayForward` is unaffected.
- **F3:** `internal/fabriccli/doc.go:43` still says the Bolt checkout "is adopted when the remote carries board history and orphan-created otherwise". The restored `--track` path matches that, and the base diff does not touch the line.
- **F6:** The added `doc.go` sentences sit right after the "weft-only" probe sentence and agree with the `TestMergeStateActive` weft cherry-pick row and the restored seam subtest.
- **F4, F2, F5:** The test changes touch no sibling text, and each now matches its base form.
- **F7:** `_lym/landing/summary.md` is outside git, so its edit is uncommitted. It is not a consistency defect.

Further calls, 2 of 6 used:
1. `sed`/`grep` for twin mentions (`ensureBoardWorktree`, owed worktree, orphan) across Go and markdown sources. My context lacked a cross-repo twin search.
2. `sed`/`git diff` of `internal/fabriccli/fabric.go` and `doc.go` against the base. My context lacked the board-adoption docs and whether the diff touched them.
