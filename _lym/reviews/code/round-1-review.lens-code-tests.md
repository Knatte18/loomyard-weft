```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test that would fail if it broke.

**Per behaviour, the test that pins it**

1. `TakeJudgeSnapshot` and `Changes` hold the decision lock while they read the checkouts, and release it. Pinned by every `TestJudgeSnapshot` row through the `unlockedReads` / `lockHeld` assertion (judgesnapshot_test.go:582).
2. When `DecisionRecord` is empty, or the seam is nil, no lock is taken. No test; this is a trivial guard.
3. The lock's error path and `ReplacedDecisionRecord`'s error path: no test. These are internal seam errors, so I raise no finding.
4. A committed replace is exempt: `DecisionRecordReplacedIsNoChange`.
5. An add after a replace is exempt: `DecisionEntryAfterAReplaceIsNoChange`.
6. An edit after a replace is still named: `DecisionRecordEditedAfterAReplaceIsNamed`.
7. An uncommitted replace is still named: `DecisionRecordReplacedButUncommittedIsNamed`.
8. `replacedDecisionRecord` in loomcli (trailer walk, path match, read at the commit): no test. Finding 1.
9. `GuardChange.Text` keeps the old text: pinned by `TestJudgeSnapshot`, which compares `texts`.
10. `GuardChange.Path`
    - Code-checkout path: pinned indirectly by `AResumeNamesWhatStillDiffers…` through the mtime suffix.
    - Records-checkout and scratch paths: no test. Finding 6.
11. `changeTime`
    - Inside-window wording: pinned by `AResumeNames…`, first call.
    - After-halt wording: pinned by `AResumeNames…`, second call.
    - Before-opened branch and the one-second grace boundary: no test. Finding 3.
    - Empty suffix for a moved HEAD or a deleted path: not pinned exactly, since `requireStuck` checks fragments with Contains.
12. `recordGuardHalt`
    - First halt records `At` and its changes: pinned by `AResumeNames…` (verify_test.go:1595).
    - An update of the changes on a later check: no test. Finding 2.
    - Clearing the halt when a check finds nothing: no test. Finding 2.
    - No new halt for a session that has not returned: no test. Finding 2.
    - No write over mismatched records: pinned by the `AHandEditOf_scope.json/judged.json` rows, because a write would error and `requireStuck` checks for a nil error.
13. The resume wording: pinned by `AResumeNames…`, which checks "the resume re-checked the judge guard" and "restored since the last check: code: a.go was changed".
    - The restored "none" branch: no test. Low value, so no finding.
14. Grouping by case, key, kind and reason
    - A keyed group with its row range, row count and first and last old/new values: pinned by `RepeatedKeyIsOneGroup`.
    - An unkeyed group ("lines #a to #b"), a split by reason, and added or removed cases kept per cell: no test. Finding 4.
15. `cappedFindings`
    - The cap, loosely: pinned by `ManyGroupsAreCapped` (a length of at most the limit plus 1000, and " more; every finding is in ").
    - Cutting a first finding that passes the limit on its own, and the exact "and N more" count: no test. Finding 5.
16. The way forward comes first: pinned by both rows of the new test (the index check).
17. The findings file holds every per-cell finding and is named by its worktree-relative path: pinned by both rows of the new test.
18. Stored-cell, stored-case and fresh-case halts: these go through the same `noMatchReason`.
    - The new test pins its shape on the fresh-cell path.
    - The older rows (`AStoredRulingOfDoesNotMatch`, the command `DoesNotMatch` row, the stored-case resume test) still pin their own fragments.
    - Nothing was loosened, so I raise no finding.
19. The batten `intentionallyNil` entries: pinned by the battencli wire-seams test's two-way check.
20. The envkit seam fills: pinned by the envkit nil-seam check.
21. Assertions at the review base: the `TestJudgeSnapshot` comparison moved to `.Text` with the same strictness. Nothing was loosened.

**Findings**

1. **MEDIUM, design.** `internal/loomcli/driftwiring.go:291-310`, `replacedDecisionRecord`.
   - Summary: the seam the replace exemption depends on has no test. Nothing covers picking the newest `Lym-Record-Replaced` trailer whose path matches the anchor-relative record path, or reading the record at that commit.
   - A wrong `rel` (one taken against the anchor instead of the record's own rel), or a wrong trailer pick, would quietly drop the exemption and bring back #146's false halt.
   - The summary admits the gap ("has no test of its own"). `TestJudgeSnapshot` uses a fake seam, so it cannot see this.
   - Evidence: driftwiring.go:291-310; `_lym/landing/summary.md`, Tests section.
2. **LOW, design.** `internal/driftengine/verify.go:765-788`, `recordGuardHalt`.
   - Summary: three branches are untested: updating a recorded halt's changes on a later check, clearing the halt when a check finds nothing, and recording no new halt for a session that has not returned.
   - `AResumeNames…` never reads `GuardHalt` after its second call.
   - Its last step ends Done through `closeJudgeSession`, which wipes the mark either way, so a broken clear or a stale `Changes` list passes. A stale list would make "restored since the last check" count from the first halt.
   - Evidence: verify_test.go:1600-1613; verify.go:437 (`closeJudgeSession`).
3. **LOW, design.** `internal/driftengine/verify.go:750-756`, `changeTime`.
   - Summary: no test reaches the "before the judge session opened" wording, and none sits at the one-second grace boundary after the halt stamp.
   - The fixture clock makes `now` equal `OpenedAt` to the second, so the first-halt row cannot tell the window's end from its start either.
   - Evidence: verify_test.go:1576-1592; verify.go:750-756.
4. **LOW, design.** `internal/driftengine/haltfindings.go:156-191`, `groupRuledCells`.
   - Summary: no test covers an unkeyed group ("lines #a to #b"), cells that differ only in reason staying apart, or added and removed cases staying one finding per cell.
   - The fixture judge writes one fixed reason for every cell, so the reason split cannot be exercised without changing it.
   - The focus directive names "rows that differ only in reason".
   - Evidence: verify_test.go:290 (fixed reason), verify_test.go:841-861.
5. **LOW, design.** `internal/driftengine/haltfindings.go:204-219`, `cappedFindings`.
   - Summary: nothing pins the cut of a first finding longer than the limit (with "…"), or the exact "and N more" count.
   - `ManyGroupsAreCapped` checks only " more; every finding is in " and a length of at most `haltFindingsLimit + 1000`. A wrong omitted count, or a cap that never cuts a single long finding, would pass.
   - Evidence: verify_test.go:860, 882-884.
6. **LOW, design.** `internal/driftengine/judgesnapshot.go:428-431, 451-459`, `GuardChange.Path`.
   - Summary: no test pins `Path` for records-checkout and scratch changes.
   - `TestJudgeSnapshot` compares only `.Text`, and the verify test dates only code-checkout files.
   - A wrong path makes `changeTime`'s stat fail, which silently drops the dating the issue asks for.
   - Evidence: judgesnapshot_test.go:575-581; verify.go:816-819.

**Tool calls**

1. Grepped `verify_test.go` and the other drift test files for no-match and findings assertions: the manifest did not hold the existing halt-reason tests.
2. Grepped `verify_test.go` for `RulingNoMatch`, case rulings and `storeRuling` to find the stored-ruling and command halt tests: the manifest had not read them.
3. Read `verify_test.go` at 645-670, 788-808 and 1036-1080 to see those tests' assertions: needed to judge whether the new reason shape loosened them.
