```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests — for each behaviour the diff adds or changes, the test row that would fail if it broke.

**Behaviours and the rows that pin them**
1. `TakeJudgeSnapshot` copies the decision record beside the snapshot: pinned by `TestJudgeSnapshot/DecisionEntryCommittedIsNoChange`. Without the copy, `appendedDecisionRecord` returns empty and the row gets "was changed".
2. `TakeJudgeSnapshot` deletes an earlier snapshot's copy before taking a new one: no row.
3. `Changes` exempts a record that only gained an added entry, committed with records HEAD moved: pinned by `DecisionEntryCommittedIsNoChange`.
4. `Changes` names the record when its identity text differs, here an edit beside an added entry: pinned by `DecisionRecordEditedBesideAnEntryIsNamed`.
5. `Changes` names the record when an entry the copy held is dropped (the `CheckAddedEntriesSubset` half): pinned by `DecisionEntryDroppedIsNamed`. Removing the subset check would make the identity texts equal and the row would fail.
6. An empty `Deps.DecisionRecord` exempts nothing: no row. Every row's fixture sets the field.
7. No copy, or the record is gone, falls through to the normal content check: no row specific to the record.
8. The read error paths in `appendedDecisionRecord` and the copy step: no row.
9. `driftDeps` sets `DecisionRecord` from `tables.decisionRecordPath`: no row. `export_test.go:12` builds `driftDeps` with an empty `driftTables{}`, and no loomcli test reads the field.
10. `ParseExpectation` refuses a `lines` entry over a baseline case whose every line is keyed: pinned by `TestParseExpectation/RangeOverKeyedCase`.
11. `ParseExpectation` accepts a `lines` entry over a mixed case (keyed and unkeyed lines): no row.
12. `ParseExpectation` accepts a `lines` entry over an added case or a baseline case with no lines: no row.
13. The `seats.go` format text: static prose, exempt.
14. The doc comments in `doc.go`, `deps.go` and `expectation.go`: prose, exempt.

**Findings**

- **MEDIUM**, design, `internal/driftengine/expectation.go:297`: neither half of the refusal's condition `unkeyed == 0 && len(occurrences) > 0` is pinned.
  - Drop `unkeyed == 0` and every mixed case would be refused. No row fails, because `expectationBaseline` has no mixed case (`expectation_test.go:15-25`).
  - Drop `len(occurrences) > 0` and a range over an added case or a baseline case with no lines would be refused. No accepted row puts `lines` on `new.txt` or `gone.txt` (`expectation_test.go:59-69`).
  - Fix: add a mixed case to `expectationBaseline` and an accepted row for a range over it, plus an accepted row for a range on the added case `new.txt`.
- **MEDIUM**, design, `internal/loomcli/driftwiring.go:249`: the `DecisionRecord: tables.decisionRecordPath` wiring has no test. Removing the line turns the #145 fix off in production while every test passes.
  - Evidence: `internal/loomcli/export_test.go:12` calls `driftDeps(..., driftTables{})`, and grep finds no `DecisionRecord` assertion in any `internal/loomcli/*_test.go`.
  - Fix: assert in a loomcli test that `driftDeps` carries the run's decision record path into `Deps.DecisionRecord`.
- **LOW**, design, `internal/driftengine/judgesnapshot.go:159-162`: no row pins that a new snapshot deletes the earlier copy. A row would take a snapshot with the record present, delete the record, take a new snapshot, recreate the record with an added entry, and expect "was added" or "was changed".
  - Without the delete, a stale copy from an earlier snapshot would exempt a record the new snapshot never held.
- **LOW**, design, `internal/driftengine/judgesnapshot.go:310-312`: no row pins that an empty `Deps.DecisionRecord` exempts nothing. Every `TestJudgeSnapshot` row inherits the field from `newJudgeFixture` (`judgesnapshot_test.go:57`).

Tool calls:
- Read the top of `expectation_test.go` and grepped its rows, to see the baseline fixture and the accepted rows, which the inherited context had only as diff hunks.
- Grepped the loomcli and driftengine tests for `DecisionRecord` and `driftDeps`, to find any test of the new wiring; the manifest held no test reads.
