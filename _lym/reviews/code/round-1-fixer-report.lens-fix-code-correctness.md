```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: trace each changed function and branch on a concrete input and check callers and tests.

No findings. I traced each change by hand and made no tool calls.

- **`Fabric.ChangedCodePaths`:**
  - Input: a `before..withCode` range carrying `.lym-anchor`, `_lym/run/status.json` and `code/a.go`.
  - `RecordsPathsAt(withCode)` returns `[.lym-anchor, _lym]`, so `DeleteFunc` leaves `code/a.go`.
  - A `ChangedPaths` failure returns early. A `RecordsPathsAt` failure is wrapped and returned.
  - The new fabric test fails if the filtering is dropped, because the extra paths would appear.
- **`landedDiffEmpty`:**
  - A nil seam returns false, as the old nil `ChangedPaths` did.
  - `ChangedCodePaths` is wired in `landingDeps`, the only production site that built `Deps.ChangedPaths`. The `ChangedPaths` assignment in `publish_integration_test.go` is a test and needs no change.
- **`RecordLandedCommit` on a re-step after a failed push:**
  - The first pass records `{L, false}`. The re-step computes true from `ChangedCodePaths(L, L)`, and the same-commit rule keeps false.
  - A different commit, such as after the archive reset, takes its own flag. A first pass recorded empty stays empty.
  - Each `TestRecordLandedCommit` row fails if `RecordLandedCommit` is swapped for `WriteLandedCommit`.
  - One change in behaviour: a non-`ErrNotExist` read error now aborts the write, where the old write would have overwritten the file. The seam only logs that error, so this is negligible.
- **`LoadConfig`:** `_lym/research` cleans to itself and hits the `_lym/` prefix check. `.` and `docs/research` still load. The new row and the old rows stay consistent.
- **Tests:**
  - The dobby row fails if the follow-up is dropped from the empty-landing line.
  - The report-passage row in `TestDarnSpec_Refusals` fails if the error from `reportPassage` is swallowed.
  - `TestDarnSpec` now asserts `docs/research/` with `LastIndex`. If other text after the Step 4 heading also contained that string, the assertion would pass even with the report passage dropped. The `reportPath` assertion would still fail in that case, so I rate it harmless.
  - `TestFinalize_RecordsWhetherTheLandedDiffIsEmpty` cannot see the records filtering, since it sits behind the seam. The fabric test covers it, so nothing is left uncovered.
