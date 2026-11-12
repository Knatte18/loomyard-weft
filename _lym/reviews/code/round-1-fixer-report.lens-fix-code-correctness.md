```yaml
lens: fix-code-correctness
outcome: reported
```

No findings. My task was to check the correctness of the changed code in `redundant.go`, `redundant_integration_test.go`, `doc.go` and `verdict.go`.

- **`map_build_failure` (F2):** With `m.Failure` non-nil, the answer map gains `{"commit":…,"error":…}`. The test's `Failure *testcover.MapFailure` field decodes that object through its `json:"commit"` and `json:"error"` tags. With `m.Failure` nil the key is absent, so the field decodes to nil. The `cov` map sets a Failure, so the happy-path `cmp.Diff` fails if the key is dropped or renamed.
- **Refusal row for a block index outside its table (F1):** The `badindex` map has `Blocks: []int{5}` against a one-entry block table. `checkMapBlocks` returns an error, so `err != nil` and the output holds "is unreadable" and "way forward" via `rebuild`. With the `checkMapBlocks` call removed, `JudgeMap` would index `m.Blocks[5]` and panic, so the row would fail.
- **Refusal row for a failed keep scan (F1):** The `badkeep` map has an in-range index, so it passes the map checks and reaches `ScanKeeps`. `keep.go:55-56` returns "has no reason" for `//testtiming:keep` with an empty reason above `func TestBadKeepMarked`. The verb then prints "the keep scan of badkeep failed" plus the way forward. With the scan error ignored, the verb would answer ok and the row would fail.
- **Tier 1 row (F5):** `TestCovSub` now carries `Tier: 1` in both the written map and the wanted candidate. Dropping the `Tier: test.Tier` assignment in `redundant.go` yields 0 and fails the diff.
- **Callers:** `runRedundant`'s signature is unchanged and the answer only gained an optional key. The `JudgeMap` edit is a comment change, and the `redundancyModule` change is a string edit.
- **Seed packages:** `badindex` and `badkeep` get generated test files with tier-0 tests, and nothing runs them. The extra `badkeep/keep_test.go` declares `TestBadKeepMarked` once, so there is no duplicate declaration. The tier-1 gate run passed, 0 failed.

Tool calls made beyond my context:
1. `grep -n -B3 -A40 "func ScanKeeps" internal/testcover/*.go` — I needed to confirm a bare `//testtiming:keep` makes the keep scan fail, which the fix text did not show.
