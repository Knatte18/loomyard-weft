```yaml
lens: fix-code-correctness
outcome: reported
```

No findings. Lens: fix-code-correctness, covering the changed code and tests.

- **`cardfixer.go`:** the notifier, `gateProgram` and the `orchcli` import are gone together. The build passed, so no caller is left broken.
- **`TestWatchRows_NoticeNeverStopsTheCall`, RefusedPost row:**
  - Call 1 has an empty history, so the visit is `{Row:"Darn-Write", Visit:0}`. The absent record reads as `{"",0}`, which differs, so the notice posts once.
  - Call 2 finds the same visit already recorded, so it posts nothing. A failed post is therefore not retried.
  - `callsAtPost` is 0 because the notice goes out before the inner call.
  - Reversing the order gives `callsAtPost == 1`, which fails the test.
  - Recording the visit only after a successful post gives `posts == 2`, which also fails it.
- **Same test, MissingStatus row:** the status read finds nothing and warns, so there are 0 posts. The inner call still runs twice and its pointer and error come back unchanged.
- **`TestNewShed_WrapsTheRowsNotifyParentOnNames`:**
  - `wire` sets `c.location = loc` (`wiring.go:439`), so the expected `RowNoticeRecord(loc, c.runID)` matches what `newShed` computes.
  - Dropping the `watchRows` call fails RecipeRow.
  - A wrong record path fails the path assertion.
  - Dropping the empty-list short cut leaves EmptyList unwrapped anyway, because `watchRows` wraps nothing for an empty list. That is behaviourally harmless, so I count it as no finding.
  - UnknownRow checks for a nil shed plus a refusal naming the row, the way forward and the recipe's rows.

Tool calls:
1. `grep "c.location = " internal/loomcli/*.go`, to confirm `wire` sets `c.location`, which my context lacked.
