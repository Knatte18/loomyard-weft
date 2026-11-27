```yaml
lens: fix-code-correctness
outcome: reported
```

Task: correctness check of the code the fix changed (`LoadState`, the new field, the `audit.go` and `runlevel.go` text, the new test rows).

Traced and holding:
- **Legacy state:** a file with only `masterNotePath: p` loads as `MasterNotePaths == [p]`, and the next save drops the old key. `keepUnknownInStruct` skips a declared key that the new encoding omits (`internal/state/keep.go:101`).
- **Load failures:** a missing file or a read error returns before the new code runs.
- **Callers:** `LoadState` is the only decoder of `State` (`state.go:219`).
- **Tests:** each new row fails with its defect in place: no legacy mapping, no `slices.Contains` skip at spawn, no empty-path skip.
- **Detail string:** finding identity uses `Key`, not `Detail`.

Findings:

1. **LOW:** `state.go` `LoadState`, `slices.Insert(st.MasterNotePaths, 0, …)`. When both keys are present, the legacy path is the newest one, but it is put first, so the "in spawn order" doc on `MasterNotePaths` does not hold.
   - **Case:** the new binary records `[a, b]`. An older binary then resumes, keeps `masterNotePaths` as an unknown key and writes `masterNotePath: c`. The next new-binary load yields `[c, a, b]`.
   - **Effect:** the allowance is unaffected, because `CheckParent` ignores order.

2. **LOW:** `state.go` `LoadState`, `st.MasterNotePath = ""` and the `!slices.Contains` condition. No added test fails if either is removed.
   - **Evidence:** the legacy-key row asserts only `MasterNotePaths` and the outcome. Without the clear, it still gets `[earlier, latest]`, while the old key survives every save.
   - **Claim left unpinned:** the field comment and `summary.md` both say a save drops the key.

Tool calls:
- `grep` of `keepUnknownInStruct` and of `State` decoders (one call): my context held only the head of `keep.go`, and no search for other typed reads of webster's `State`.
