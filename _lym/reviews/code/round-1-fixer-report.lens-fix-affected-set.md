```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the F1 and F2 searches to check the fix covers every member of each set.

**F1** (references to the moved test file): no findings. A repo-wide grep for `smoke_lifecycle_test.go` and `smoke_lifecycle_windows_test.go` finds one stale-looking reference, and it is not stale.
- `internal/loomcli/smoke_test.go:4` cites `smoke_lifecycle_test.go` as an example of the smoke suite's conventions. That file still exists and still holds the cross-platform lifecycle tests, so the reference is valid.
- `internal/reedengine/doc.go:485` now names the Windows file.
- `TestSmokeRemoveLastStrandThenAddRunsTheNewCommand` is cited by name only in `doc.go`, so no other site needs the path.

**F2** (`PATTERN-test-speed` entry): no findings. The rule is carried in `PATTERN.md:60`, the one place that holds the entry line. The other `PATTERN-test-speed` mentions are plain citations. They are the tier-tag and LLM-tier test headers, `docs/benchmarks/*`, and `docs/research` bench prompt snapshots. The snapshots are historical records and need no update.

Tool calls:
1. Grep for both smoke_lifecycle file names, plus `ls internal/reedcli`: to find every reference to the moved test's old and new path, which my context lacked.
2. Grep for `PATTERN-test-speed` across `.md` and `.go` files: to find every site that restates the entry, which my context lacked.
