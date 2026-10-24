```yaml
lens: code-tests
outcome: reported
```

**Lens code-tests: one line per behaviour, with the test that pins it**

- **`RecoveryRetry` returns true at zero counted recoveries with an empty start.** Pinned by the new row "a dead recovery a batch-start reset cleared retries" in `recoverbatch_test.go:1589`. It uses `deadRecovery(0, "")` and wants `(true, 0)`. Reverting the early return makes it fail. The row also leaves the count at zero, as the acceptance criteria ask.
- **The commit rule is kept at one counted recovery.** Pinned by existing rows that were not edited. They cover commits past the start (`recoverbatch_test.go:1580`), none of its own (`:1597`) and a fork's earlier commits (`:1604`). They also cover a HEAD moved off the start, merge-ins only, and a failed git read with its Warn line.
- **The cap still refuses.** Pinned by the existing "a dead recovery at two recoveries does not retry" row (`:1650`). The new zero branch sits after the cap check, and this row confirms the order.
- **The kind, terminal and status checks still apply.** The existing stuck and failed rows cover status. The kind and terminal checks, and the zero branch for them, have no row. At zero recoveries, a non-terminal record, a fork record and a nil batch are not pinned as false. This is an uncovered branch ordering, not a missing behaviour; I report it as LOW below.
- **"A record with no start does not retry" is still meaningful.** It was already `deadRecovery(1, "")` (`:1613`), so the diff left it as it was. It now pins that an empty start only refuses at one counted recovery.
- **`begin-batch`'s report-present remedy names `recover-batch` for a reset dead batch.** Pinned by the new `beginbatch_test.go:194` row, which asserts the new wording and `wantNotText` excludes "exhausted" and "record-batch". The existing "terminal dead recovery that committed" row at `:251` asserts only the prefix "once more", so it still holds under the new wording.
- **`begin-batch` still refuses a dead recovery that earns no retry.** The fixture at `beginbatch_test.go:187` was changed to `Recoveries: 1`. The decision record licenses this, because the old zero-count fixture is now the retryable case. The row's meaning is kept.
- **`status` reports `recovery_retry: true` and `recoveries: 0` for a reset dead batch.** Pinned by the new `cli_test.go:841` row. The worktree is no repository, so a true verdict also proves no git read happens on that path.
- **The `recover-batch` terminal envelope at zero recoveries.** No row pins it at zero. `verbs_test.go:1123` pins retry true only at `recoveries: 1`. The envelope reads the same `RecoveryRetry` as `status`, so it has no logic of its own; the engine row covers the verdict. I did not count this as a coverage gap.
- **Wording changes in the stencil, `doc.go`, the `recover-batch` long help and the `state.go` comment.** They are static prose and exempt from coverage.

**Test economy**
- The engine row, the `begin-batch` row and the `status` row each cover a different reader of the verdict: the verdict itself, the remedy wording, and the JSON fields. I found no duplicates. The `status` row is the closest to redundant with the engine row, but it pins the `recoveries` and `recovery_retry` fields and the no-git path, which the engine row does not.
- No per-helper test was added, no stencil prose assertion was added, and no review-fix test appears.

**Finding**

- **LOW, class scope, `internal/websterengine/recoverbatch_test.go:1570` (`TestRecoveryRetry`).** The zero-count early return is new, and its position after the kind, terminal and status checks is what stops a non-recovery or non-terminal record from reading true. No row has `Recoveries` 0 with `Kind` "fork" or `Terminal` false. Moving the `recoveries == 0` check above the kind check would pass every current row, and so would dropping the kind and terminal checks for zero-count records. The fix is to add one or two rows to this table, for example `deadRecovery(0, "")` with `Kind` set to "fork" and with `Terminal` false, both wanting `(false, 0)`. The fixture is at `:1700`, and a row's `setup` can mutate the returned record, as the stuck and failed rows do.

**Tool calls (1 of 6)**
1. Read `recoverbatch_test.go:1566-1700`, `cli_test.go:800-842` and a grep for retry assertions across `webstercli` and `websterengine` tests. My context held the diff hunks but not the surrounding rows of the existing tests.
