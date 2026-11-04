```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code changed for F1–F4 (`HoldsFile`, `finishDone`/`commitRecords`, and the added tests) through the fix-code-correctness lens.

no findings

Traces I ran:
- **`shedrun.HoldsFile`:** an absent path fails `os.Stat` and returns false. A plain file fails the `IsDir` check and returns false. A directory with a file beneath it is found by the walk and returns true. The callers keep the semantics of the removed `holdsFile`: `statusCommitPathspec` (twice), `planCommitPathspecWith` and `landingdeps.go`, where `shedrun` was already imported. `wiring.go` still uses `os`, `fs` and `filepath` elsewhere, and the build and vet pass.
- **`finishDone`:**
  - A failing first `CommitRecords` still returns the wrapped error, as before.
  - Without friction, the function returns nil after one commit.
  - With friction `failed`, `handBack` runs and `d.exit` is reset to `ExitDone`.
  - If `handBack` errors, it returns before the second commit. Otherwise the second commit runs, and its error is wrapped by the shared `commitRecords`.
  - With `CommitRecords` nil, both calls are no-ops, so the `noCommit` rows are unaffected.
- **Updated `sheddriver` row:** it now expects `commit-records` twice. With the old single-commit code it would fail.
- **`TestBattenRecordsCommitSeams`:**
  - Removing the Park commit leaves `messages` empty, so the test fails.
  - Turning the warning into a returned error makes `Park` return non-nil, so the test fails.
  - Skipping the marker write fails the marker read.
  - A `CommitRecords` that swallows the commit error fails the error-parity check.
  - The closures are called sequentially, so the shared `messages` has no race.
  - The success case returns `committed` false, so `Spawn` is never reached. That is fine, because the finding did not ask for it.
- **`TestHoldsFile`:** the `PlainFile` row fails against the old `dirHoldsFile`, which returned true for a plain file, so the F2 regression is pinned.

Further tool calls: none.
