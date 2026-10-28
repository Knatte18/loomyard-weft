```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test that pins it, or none.

**Per behaviour, the test that pins it**

1. The note is told whenever a Darn-Write history entry exists, and is empty when none exists.
   - Pinned by `TestRenderPriorWork` (`internal/loomshed/darnwrite_test.go:210-214`). The row "an earlier entry with no kept findings" covers the told case. The rows "nothing to name" and "findings of an entry the read did not find are ignored" cover the empty case.
   - `TestDarnWrite_Call`, row "prior work names the earlier outcome and its kept findings" (`:70-83`), covers it through the producer.
2. The note names the earlier outcome ("That spawn ended stuck." and "…done."): pinned by the same rows as 1, through exact string equality.
3. The note names the findings path when one is set and omits that line otherwise: pinned. The `TestDarnWrite_Call` row has a path, and the `TestRenderPriorWork` row has none.
4. The note carries the decisions-win line: pinned by exact equality in both tests.
   - The expectation interpolates `decisionsWinLine` itself, so a wording regression in the constant would not fail either test.
   - The wording is only static prose, so I don't count this as a finding.
5. `latestDarnOutcome` sets `FindingsPath` only when the file exists: pinned by `TestLatestDarnOutcome` (`internal/loomcli/wiring_darn_test.go:408-478`).
   - The row "a resumed run, its error cleared, names the kept findings" covers the file-exists case.
   - The row "latest Darn-Write entry of a run that moved on, with no findings kept" covers the absent-file case.
   - The row "a Darn-Write reroute entry is skipped…" covers the skip-reroute path with findings kept.
6. The halt-reason read is dropped: pinned by the resumed-run row, a status of `StateRunning` with an empty error that still reports found. The stale-error row, whose status carries `Error: "stale"`, also asserts no `Reason` field exists.
7. `DarnGateReprompt` and every gate entry carries it: pinned by `TestDarnWriteEntry_HappyPath` (`internal/shedrecipe/entries_darnwrite_test.go:97-108`).
   - It loops over every entry of the built table's gate (both `verify` and `description`).
   - It asserts the entry is non-nil, names the findings path, names `env.DecisionRecordPath`, contains "Added after Discussion", and has no newline.
8. The removed `Reason` rows ("outcome without a reason", "reason only", "reason … ignored") leave no gap. Their behaviour is gone, and the replacement rows cover the new keying.

**Findings**

- LOW, design, `internal/loomcli/darnwiring.go:135-139`: the branch that returns a non-not-exist `os.Stat` error has no test row. Every `TestLatestDarnOutcome` row either has the file or lacks it with `ErrNotExist`. The branch is new, and it turns a stat fault into a hard error from the producer. A row with a path whose parent is a regular file would give `ENOTDIR`, which is not `ErrNotExist`, and would exercise it portably.
- LOW, design, `internal/loomcli/darnwiring.go:156`: the `darnDeps` closure passes `shedrun.GateFindingsFile(l, runID, loomshed.NameDarnWrite)` into `latestDarnOutcome`. No test reaches `darnDeps().LatestOutcome` (grep found no caller in `internal/loomcli/*_test.go`). If the closure passed a different path from the one `Env.GateFindingsFile` gives the row, the note would silently drop the findings line and no test would fail. `TestLatestDarnOutcome` passes its own path, so it cannot catch this.

Nothing else is uncovered.

**Tool calls**
1. `grep darnDeps(\|LatestOutcome()` in `internal/loomcli/*_test.go`, plus `sed -n 470,490p` of `wiring_darn_test.go`: my context lacked the tail of `TestLatestDarnOutcome` and whether any test calls the `darnDeps` closure.
