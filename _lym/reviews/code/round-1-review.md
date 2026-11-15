---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/boardengine/sketch_test.go (TestUpsertTaskReportingRecordWarning table); guards at internal/boardengine/decision.go:91-92
    summary: the isDone and recordStored guards of bodyEditRecordWarning have no test row that fails if either guard is dropped
    fix: in TestUpsertTaskReportingRecordWarning add two silent rows. One edits the body of a done discussed entry whose record is stored; set it done through the board's status path before the write under test. The other deletes the entry's decision-<slug>.md file from boardPath after StoreDecisionRecord, then edits the body of the still discussed entry. Both expect an empty RecordWarning and the body written. Add the setup the rows need as table fields beside storeRecord and clearStamp.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/doc.go:700
    summary: the doc still lists UpsertTaskSiblings among the siblings-returning methods, but the diff removed it in favor of UpsertTaskReporting
    fix: on that line replace UpsertTaskSiblings with UpsertTaskReporting.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: plugins/ly/skills/board/SKILL.md:134
    summary: the board skill documents the warning key for promote only, so it does not tell an agent that upsert of a body edit on an entry with a stored record now warns
    fix: beside the promote warning text at lines 113-135, add that upsert without --record-file adds a warning key when it edits the body of an open entry whose record is stored and was not stored against the new body. State that the warning names upsert --record-file and lym loom decision add, and that the write still lands.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardcli/verbs_test.go (stepUpsertRecordFile)
    summary: nothing asserts that an upsert with --record-file carries no warning key, although that path deliberately leaves the warning out
    fix: in stepUpsertRecordFile, capture the result map of the upsert with --record-file and add an assertion that result["warning"] is absent. Do not add a new test function.
---

### [MEDIUM] Guards of bodyEditRecordWarning with no failing row

**Location:** `internal/boardengine/sketch_test.go` (`TestUpsertTaskReportingRecordWarning`), guards at `internal/boardengine/decision.go:91-92`
**Issue:** The table pins the title-only edit, the restored body, the unstamped record, the undiscussed entry and the plain warning.
No row reaches the `isDone(after)` guard or the `recordStored` guard.
The undiscussed row returns at `!after.Discussed` first, so deleting `recordStored` from the condition fails no test.
The board entry's expected cases include a landed entry and a missing record file.

### [LOW] Stale method name in the boardengine package doc

**Location:** `internal/boardengine/doc.go:700`
**Issue:** The line still reads "SetDeps, UpsertTaskSiblings, UpsertTasksBatch and MergeTasks return the siblings."
The diff removed `UpsertTaskSiblings`, and a grep finds no other reference to it.

### [LOW] Board skill omits the new upsert warning

**Location:** `plugins/ly/skills/board/SKILL.md:134`
**Issue:** The skill describes the `warning` key for `promote` only.
An agent that edits the body of a discussed entry reads that the upsert is silent about the record, which is the gap the board entry reports.

### [LOW] The record-file upsert's missing warning is not asserted

**Location:** `internal/boardcli/verbs_test.go` (`stepUpsertRecordFile`)
**Issue:** The `--record-file` path leaves `RecordWarning` empty because it replaces and stamps the record itself.
No assertion fails if that path starts emitting the warning.

## Rejected

- Hand-built envelope in `internal/boardcli/cli.go`: the shape matches `outputSuccessWithTask` (`task`, plus `notice` only when non-empty), and `outputSuccessWithTask` keeps its other callers.
- The `existed` guard in `UpsertTaskReporting` has no distinguishing test: with an empty `before`, the `!Discussed` guard already gives silence, so no test could observe it.
- Warning text and the `--record-file` path it names: `lym board get --record` exists at `internal/boardcli/cli.go:918`, and `upsert --record-file` is the path that works before a worktree exists.
- Test overlap among `verbs_test.go`, `cli_test.go` and `siblings_test.go`: the edits are a step rename, a shared assertion and a call rename, and `TestUpsertTaskReportingRecordWarning` (reported unique) pins the engine behavior, so none duplicates another.
- Correctness of every changed branch: the code-correctness lens ran its cases and found no defect, and my own read agrees.
