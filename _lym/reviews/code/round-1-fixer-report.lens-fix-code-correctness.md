```yaml
lens: fix-code-correctness
outcome: reported
```

no findings

I walked each changed branch with a concrete input, including the failure of each call it makes:

- **F8 (`RowsHoldSplitPerch`)**: `RaddleProposalRel` is a `filepath.Join` result, so it is already clean and the `filepath.Clean` comparison is sound. `./_lym/raddle/proposal.yaml` cleans to the proposal path, so the new row passes. Without `Clean` the row fails. A non-string or empty path cleans to `.`, which never equals the proposal. `bouncerEngine` exists in the package, so the new row compiles.
- **F1 rows**:
  - Perch at Raddle-Write: the row differs from the current row, so `perchFilingRefusal` returns empty and the decision appends. Dropping the `currentRow != row` guard would make this row refuse.
  - Perch with no Raddle-File row: `hasRow` is false. Dropping `!hasRow` would match an empty row against an empty current row and refuse.
  - The three error rows each reach `refuse("read the run's raddle filing: …; way forward…")` through the new error fields on the fake.
- **F3, F2 and F4**: each fails if its production line is removed.
  - Dropping the `Perch:` assignment fails the PerchModeState row.
  - Dropping the created-stamp check fails the earlier-entry row.
  - Dropping the Sketch check fails the another-sketch row.
  - Dropping the board-gone tolerance fails the gone-entry row.
  - Skipping the perch store in `StoreCold` fails the perch row.
- **F5**: the failing split-perch read returns before any state update, so state stays zero and the inner producer never runs. The `append(tt.want, "re-step")` call copies because the slice literal has no spare capacity. Ignoring the `SplitPerch` error would let the call proceed and fail the row.
- **F6 and F7**: text only. Tier 0 passed with no failures, so no test pins the old comment or heading.
- **Callers**: no signature changed, so no caller is affected.

Tool calls:
1. `grep` of `RaddleProposalRel` in `internal/loomengine/raddlepaths.go` and `bouncerEngine` in `internal/loomrecipe`, because my context lacked both definitions.
