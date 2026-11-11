---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/loomcli/cli_test.go:40
    summary: the loom pre-run's skip of the parent-review group has no test row
    fix: In internal/loomcli/cli_test.go add parentReviewGroup to the group slice that TestResolvePersistentPreRun_SkipsSlugGroups iterates (now circling, decisionGroup, peersGroup), and name the parent-review group in the comment at line 29.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/parentreview_test.go:75
    summary: no verb-level test shows a findings file's severity, class and change reach the written report
    fix: In the row "a findings file answers with its findings" of TestParentReviewAnswerVerb, read the report at deps.files.ReportPath(1) after the call and compare its findings with the file's items, ids P1 and P2, severity MEDIUM and class design on the first and none on the second, so a dropped mapping in readParentFindings fails the test.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/parentreview_test.go:96
    summary: the caller guard's admission of the resolved parent orch and its refusal when the parent cannot be resolved are untested
    fix: In TestParentReviewAnswerVerb add one row with setup strandName = parentReviewParent over an open ask expecting state done, and one row with a deps.parent that returns an error and a strandName that is neither empty nor the ask's strand, expecting a refusal holding "could not be resolved" and parentReviewWayParentAnswers with the ask left open.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/parentreview_test.go:92
    summary: the empty findings file, malformed YAML, an unreadable path and a whitespace-only change have no rows
    fix: In TestParentReviewAnswerVerb add rows for a findings file that is empty, one holding "[", a --findings-file path that does not exist, and an item with change "  ", each expecting a refusal that holds parentReviewWayFindingsFile and leaves the ask as it was. The empty-file row needs the findings text to reach the file even though it is empty, so write the file whenever the row sets a flag instead of testing tt.findings != "".
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/burlerengine/parentreview_test.go:52
    summary: an ask in a state this binary does not know is never shown to stay unmoved
    fix: In internal/burlerengine/parentreview_test.go add a row to TestAnswerParentAsk and one to TestParentAsks_RecordAndMoveForward with an ask of State "future-state" and a deadline at or before now. Assert the state stays "future-state", the answer is refused with ParentAnswerEnded, and the expiry leaves it unchanged.
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/parentreview_test.go:191
    summary: the await's wait-cut-short refusal on a sleep error has no row
    fix: In TestParentReviewAwaitVerb add a row over an open ask whose deps.sleep returns context.Canceled, expecting an error envelope that holds "the wait was cut short" and parentReviewWayAwaitRetry. Give the table a field that selects that sleep, since the shared fake sleep always returns nil.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/burlerengine/parentreview_test.go:240
    summary: TestAnswerParentAsk_RacingTheDeadline is a contained candidate that pins what its covering tests cannot see
    fix: Keep the test and add the line "//testtiming:keep pins that a concurrent answer and expiry leave exactly one end state under the one lock" directly above its func TestAnswerParentAsk_RacingTheDeadline line. TestAnswerParentAsk and TestParentAsks_RecordAndMoveForward run each move alone and never assert the two together.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/burlerengine/parentreview.go:237
    summary: AnswerParentAsk returns the past-deadline refusal ahead of a failed record write, so a timeout that was not persisted is reported as set
    fix: In AnswerParentAsk check err before refusal, and return the wrapped write error when UpdateJSON failed, so the refusal is returned only when its state change was written. Keep the early return for refusals whose mutate returns errParentAskUnchanged, which write nothing.
---

### [MEDIUM] The pre-run skip of the parent-review group is untested

**Location:** internal/loomcli/cli_test.go:40
**Issue:** `inReviewGroup` now skips `parentReviewGroup` (`internal/loomcli/cli.go`), and commit `f0760833` exists for exactly that skip. `TestResolvePersistentPreRun_SkipsSlugGroups` still iterates only `circling`, `decisionGroup` and `peersGroup`. Removing the skip would arm the group's slug as a run-id and break both verbs, and no test would fail. The affected-set lens found the same gap.

### [MEDIUM] The findings file's severity, class and change are not shown to reach the report

**Location:** internal/loomcli/parentreview_test.go:75
**Issue:** D4's acceptance criterion says each finding keeps its given severity and class, with ids P1 and P2 in the file's order. The verb test asserts only the count `2` and the state `done`. `TestAnswerParentAsk` builds `ParentFinding` values directly, so a dropped mapping in `readParentFindings` fails no test.

### [MEDIUM] The caller guard's parent-admission and unresolved-parent branches are untested

**Location:** internal/loomcli/parentreview_test.go:96
**Issue:** The rows cover the operator's shell, the ask's strand and one stranger. No row runs a named caller equal to the resolved parent orch, which is D4's primary admitted caller, and no row runs a named caller when `deps.parent` fails. Either branch of `parentAnswerCallerRefusal` could break with the suite green.

### [LOW] Empty, malformed and unreadable findings files, and a whitespace-only change, have no rows

**Location:** internal/loomcli/parentreview_test.go:92
**Issue:** `readParentFindings` refuses each of these by naming the file's shape, per D4. Only the empty list, a missing change, a bad severity and a bad class are reached. The empty file takes the `io.EOF` branch, which no row reaches.

### [LOW] The unknown-state invariant has no test

**Location:** internal/burlerengine/parentreview.go:42
**Issue:** `Ended` counts a state this binary does not know as ended so that an ask in it is never moved. This is the invariant the round's focus asks about, and no test records an ask in such a state.

### [LOW] The await's wait-cut-short refusal has no row

**Location:** internal/loomcli/parentreview_test.go:191
**Issue:** When `sleep` returns an error, as it does for a cancelled context, the await refuses with `the wait was cut short`. The shared fake sleep always returns nil, so the branch is never run. The code is correct, as the correctness lens ran it; the gap is coverage.

### [LOW] TestAnswerParentAsk_RacingTheDeadline needs a keep marker, not a fold

**Location:** internal/burlerengine/parentreview_test.go:240
**Issue:** The redundancy report lists the test as contained, covered by `TestAnswerParentAsk` and `TestParentAsks_RecordAndMoveForward`. Their assertions are sequential: each runs one move alone. None asserts that an answer and an expiry running together leave exactly one end state, which is D5's acceptance criterion and what this test pins. It is accepted, and per the rubric it is marked `//testtiming:keep`.

### [LOW] A past-deadline timeout that failed to persist is reported as set

**Location:** internal/burlerengine/parentreview.go:237
**Issue:** On the past-deadline path the locked update returns `nil`, so the record write can still fail. `AnswerParentAsk` checks `refusal != nil` before `err`, so the caller gets the past-deadline refusal, which says the ask is now `timeout`, when the write failed and the ask stays `open`. A later await or answer re-derives the timeout, so the effect is a misreported error and not a lost state.

## Rejected

- Report left on disk when the record write fails after the report is written (code-correctness F1): the ask stays `open` and no reader opens a report whose ask is not `done`; a later successful answer overwrites it at the same path. The diff does not claim otherwise, so nothing is gained by moving the write.
- Scope: D4's copy of a `done` report beside the accepted review and D5's review-gate close of an ask the chair left open are not in the diff. The record's `## Goal` delivers the ask record, the keys, the cap and the two verbs, and its `## Scope` names only `internal/loomcli`, `internal/burlerengine`, `internal/loomengine` and `internal/lymdirs`. Both deferred steps need `internal/shedadapters`, so the omission is licensed, and the change description names it.
- Docs: `docs/overview.md`, `internal/burlerengine/doc.go` and `internal/loomcli/doc.go` each carry the new verbs and the ask record, so no rule-required doc is missing.
- `CapParentReviewWait` and `chairTimeout`: the arithmetic matches D5 (chair timeout less start allowance, one await cap and the 5-minute margin, the lesser of the key and the room, a zero return with a reason naming both keys when the room is zero or less), and `CapAssistTimeout` behaves as before.
- Config keys: both are category `none`, defaulted in the template, validated with a refusal naming the key, and both template comments say they are settable through `run_settings`.
- Commits: none of the five commit messages carries a `Pattern-Departure:` trailer, and the diff declares no PATTERN change.
