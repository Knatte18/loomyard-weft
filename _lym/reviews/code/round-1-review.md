---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/driftengine/judgesnapshot.go#appendedDecisionRecord
    summary: The replace base is the newest replace commit of any age, so a judge commit that rolls the decision record back to a replace made before the snapshot, dropping a parent entry added since, is exempt and goes unnamed.
    fix: Use the replace base only for a replace committed after the snapshot's records HEAD. Either pass s.Records.Head to Deps.ReplacedDecisionRecord and have the loomcli wiring (internal/loomcli/driftwiring.go#replacedDecisionRecord) return found false unless the trailer commit lies after that head (compare TrailerCommit.Position against the head's position, or require the commit among those since the head), or have appendedDecisionRecord check the replace commit against the snapshot head itself. Keep the snapshot copy as the only base when no such replace exists. Add a TestJudgeSnapshot row in which a replace R and an added entry predate the snapshot (snapshot copy is R plus the entry, f.replaced is R), and the judge commits R; want records decision-record was changed.
    origin: lens:code-correctness
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/driftengine/paths.go:69
    summary: Findings repeats the logs directory literal that LogsDir uses on the line above, which PATTERN-code-conventions forbids, and a rename of one would move the findings file out of the LogsDir guard exemption.
    fix: In internal/driftengine/paths.go declare the logs segment once (a constant such as logsDirName beside driftDirName) and the findings file name as a constant beside expectedFileName and judgementFileName (such as findingsFileName = "verify-findings.md"). In NewPaths compute logsDir := filepath.Join(durable, logsDirName) and set LogsDir to logsDir and Findings to filepath.Join(logsDir, findingsFileName).
    origin: lens:code-affected-set
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/driftengine/haltfindings.go#groupRuledCells
    summary: A group's first and last old and new values come from the cells' input order, which for fresh rulings is the judge's order, while the row range comes from the minimum and maximum row, so the reported endpoint values can belong to other rows than the range's ends.
    fix: In groupRuledCells sort each group by rowOf before taking first and last (slices.SortStableFunc on the group by rowOf), so first is the cell at low and last the cell at high. Add a TestDriftVerify findings-table row whose judge rules the cells in shuffled order and assert the range and the endpoint values of its first and last rows.
    origin: assist:assist
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/loomcli/driftwiring.go#replacedDecisionRecord
    summary: The production seam that picks the newest Lym-Record-Replaced trailer naming the record and reads the record at that commit has no test, so a wrong relative path or trailer pick would silently drop the replace exemption and bring back the issue 146 false halt.
    fix: Add a tier1 loomcli test (hermetic git through the existing loomcli test fixtures) that commits two replace trailers for the decision record, a newer replace trailer naming another path, and a plain commit, then calls replacedDecisionRecord and asserts the bytes of the newest matching replace and found true, and found false on a records branch with no matching trailer. Cover the after-snapshot restriction F1 adds in the same test.
    origin: lens:code-tests
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/driftengine/verify_test.go#TestDriftVerify
    summary: The grouping rows cover only one keyed group with one shared reason, so nothing pins that cells differing only in reason stay apart, that an unkeyed group words its lines range, or that added and removed cases stay one finding per case.
    fix: Extend the a-halt-over-many-ruled-cells table in TestDriftVerify. Let the fixture judge take a per-cell reason (for example a map from case and key to reason, defaulting to the current fixed text). Add a row with one case and key whose cells carry two reasons and assert two findings. Add a row of unkeyed changed lines and assert the lines #a to #b wording.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/driftengine/haltfindings.go#cappedFindings
    summary: The cap leaves the semicolon separators, the ellipsis and the and-N-more suffix outside haltFindingsLimit, and no test pins the cut of an over-long first finding or the exact omitted count.
    fix: In cappedFindings count each kept finding's separator in length and reserve the suffix, so the joined text stays within haltFindingsLimit. In the TestDriftVerify findings table, assert the exact and-N-more count in ManyGroupsAreCapped and that the findings part of the reason is at most haltFindingsLimit, and add a row whose single finding passes the limit and assert it ends with the ellipsis.
    origin: lens:code-correctness
  - id: F7
    severity: LOW
    class: design
    location: internal/driftengine/verify.go#recordGuardHalt
    summary: No test checks the recorded halt after a resume, so an unchanged At with updated changes, the clear when a check finds nothing, and no halt for a session that has not returned are all unpinned.
    fix: In AResumeNamesWhatStillDiffersWhatWasRestoredAndWhenEachFileChanged, after the second call assert f.scope().JudgeSession.GuardHalt has At equal to the first halt's and Changes equal to only the b.go change. Before the final requireDone, run a pass after the restore that stops for another reason or read the mark on the pass that settles the session, and assert GuardHalt is nil.
    origin: lens:code-tests
  - id: F8
    severity: LOW
    class: design
    location: internal/driftengine/verify.go#changeTime
    summary: The before-the-judge-session-opened wording has no test that reaches it.
    fix: Add a step to the AResumeNames subtest, or a row of the guard table, where the judge edits a file during the session and sets its mtime to before the mark's OpenedAt with os.Chtimes, and assert the before the judge session opened at wording.
    origin: lens:code-tests
  - id: F9
    severity: LOW
    class: design
    location: internal/driftengine/verify.go#settleOpenSession
    summary: A guard halt over a session whose judge never returned records nothing on the mark, so each later resume repeats the first halt's text without the still-differs and restored lines that issue 146 asks a resume to give.
    fix: In recordGuardHalt, for a check of a session that has not returned, record the changes found with an empty At (and make changeTime treat an empty At as no halt end), so the next check reads mark.GuardHalt and words the resume lead with what was restored.
    origin: lens:code-correctness
  - id: F10
    severity: LOW
    class: design
    location: internal/driftengine/verify_test.go#TestDriftVerify
    summary: The findings-file assertion only counts bullet lines, so empty or duplicated bullets would pass, although the file is the one place the capped cells are kept.
    fix: In the findings table of TestDriftVerify, assert that the findings file holds the exact describeRuledCell line of a cell the capped reason leaves out, such as the last row of RepeatedKeyIsOneGroup and the last case of ManyGroupsAreCapped.
    origin: assist:assist
  - id: F11
    severity: LOW
    class: design
    location: internal/driftengine/judgesnapshot.go#checkoutChanges
    summary: No test pins GuardChange.Path for a records-checkout or scratch change, so a wrong path would make changeTime's stat fail and silently drop the dating for the decision record, the very file issue 146 is about.
    fix: In TestJudgeSnapshot also compare each change's Path with the want, adding a want path per row (the records-checkout file for a records change, the scratch file for a scratch change, empty for a deletion or a moved HEAD).
    origin: lens:code-tests
---

### [BLOCKING] A replace made before the snapshot exempts a judge's removal of a decision entry

**Location:** internal/driftengine/judgesnapshot.go#appendedDecisionRecord

**Issue:** `appendedDecisionRecord` accepts the record when it differs only by added entries from either the snapshot copy or the record as the newest replace commit wrote it.
`replacedDecisionRecord` in `internal/loomcli/driftwiring.go` returns the newest replace trailer naming the record, whatever its age.
`CheckAddedEntriesSubset(now, base)` holds the record only to the base's entries.
Take a replace R committed before the snapshot, then a parent `decision add` of E, then a snapshot holding R plus E.
If the judge commits R, the snapshot base fails because E is gone, but the replace base R matches, so the removal is exempt.
Before this diff that removal was named.
The new `doc.go` text still claims an edit or removal of an existing entry is named, and its only stated bound is a commit carrying a replace trailer.
The focus directive's first question, whether the guard still halts on a judge change to the decision record, is answered no for this case.

### [BLOCKING] The findings path repeats the logs directory literal

**Location:** internal/driftengine/paths.go:69

**Issue:** `Findings: filepath.Join(durable, "logs", "verify-findings.md")` repeats the `"logs"` that `LogsDir` uses on the line above.
`PATTERN-code-conventions` requires a name or path that can go stale to be one named constant, never a repeated literal.
The judge guard exempts the findings file only because it lies under `LogsDir`, so renaming one literal and not the other would make the findings file read as a judge edit.

### [MEDIUM] Group endpoint values follow input order, not the row range

**Location:** internal/driftengine/haltfindings.go#groupRuledCells

**Issue:** `low` and `high` are the minimum and maximum `rowOf`, but `first` and `last` are `group[0]` and `group[len(group)-1]` in input order.
`storeRulings` collects the cells in `session.Judgement.Cells` order, which the judge chooses, and `ParseJudgement` does not sort it.
For rows ruled in the order 3, 1, 2, the finding names rows 1 to 3 but gives the values of rows 3 and 2.

### [MEDIUM] The production replace-trailer read has no test

**Location:** internal/loomcli/driftwiring.go#replacedDecisionRecord

**Issue:** `TestJudgeSnapshot` feeds the replaced bytes through a fake seam.
No test covers the trailer walk, the anchor-relative path match against `parseRecordReplaced`'s path, or the read through `AnchoredFileAt`.
The change description says this wiring has no test of its own.
The issue 146 fix rests on it.

### [MEDIUM] Grouping by reason and unkeyed groups are unpinned

**Location:** internal/driftengine/verify_test.go#TestDriftVerify

**Issue:** The fixture judge writes one fixed reason for every cell, and the new rows use either one keyed series or one cell per case.
Dropping the reason from `rulingGroupKey` or breaking the `lines #a to #b` wording would pass every test.
Issue 147 names the rationale as part of the grouping, and the focus directive asks about rows that differ only in reason.

### [LOW] The cap leaves formatting outside its budget and its edges are untested

**Location:** internal/driftengine/haltfindings.go#cappedFindings

**Issue:** `length` adds each finding but not the `"; "` joiner.
The ellipsis after a cut first finding and the `and N more` suffix also sit outside the limit.
`ManyGroupsAreCapped` checks the reason against `haltFindingsLimit + 1000` and only the ` more; every finding is in ` fragment.
No row has a single finding longer than the limit.

### [LOW] The recorded guard halt is not checked after a resume

**Location:** internal/driftengine/verify.go#recordGuardHalt

**Issue:** The resume subtest reads `GuardHalt` only after the first call.
Its final pass ends Done through `closeJudgeSession`, which replaces the whole mark.
A broken update of `Changes`, or a broken clear, would therefore pass.
A stale `Changes` list would make the next "restored since the last check" count from the first halt.

### [LOW] The before-session dating is unreached

**Location:** internal/driftengine/verify.go#changeTime

**Issue:** The new subtest covers a file modified inside the window and one modified after the halt.
`AChangeMadeBeforeTheSpawnIsNoJudgeEdit` makes no guard change, so it never reaches the `before the judge session opened` branch.

### [LOW] A halt over a session that never returned keeps repeating its first text

**Location:** internal/driftengine/verify.go#settleOpenSession

**Issue:** `settleOpenSession` calls `changedSince` with `returned` false.
With no recorded halt, `recordGuardHalt` takes its `default` branch and writes nothing.
A session whose judge did not end Done, for example after a crash or a failed judgement entry, halts on each resume with the plain "the judge session changed what it may not" lead.
That lead never names what was restored.

### [LOW] The findings file is checked only by its line count

**Location:** internal/driftengine/verify_test.go#TestDriftVerify

**Issue:** `strings.Count(string(data), "\n- ")` counts bullets.
A file of empty or repeated bullets passes.
The rows never check that a cell the capped reason leaves out is in the file.

### [LOW] GuardChange.Path is pinned for code-checkout files only

**Location:** internal/driftengine/judgesnapshot.go#checkoutChanges

**Issue:** `TestJudgeSnapshot` compares only `Text`, and the verify subtest dates only code-checkout files.
The records-checkout path, which is where the decision record sits, and the scratch path are not asserted.

## Rejected

- The scope lens's note that `haltReasons` appends the uncapped loosened-check reason after the capped no-match reason: issue 147 is about the findings over judge rulings, and the loosened-check reason is a separate halt cause that the diff does not change.
- The correctness lens's NIT that the cap counts bytes rather than characters: `len` on the ASCII-dominated findings text is the repository's usual measure, and the separator part of that NIT is kept in F6.
- The assistant's view that every test gap is MEDIUM: the grouping-by-reason and replace-wiring gaps are MEDIUM because they pin what the issues ask for. The before-session dating and findings-file content gaps cover narrow branches, so they stay LOW.
