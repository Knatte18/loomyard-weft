---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/driftengine/judgesnapshot.go:383
    summary: A replacement predating the snapshot exempts a later judge deletion of an existing decision.
    fix: Pass the snapshot records HEAD into the replacement lookup and accept only replacement commits after that HEAD. Keep the snapshot comparison authoritative when no such replacement exists. Add a regression in TestJudgeSnapshot for replacement R, addition A, snapshot of R+A, then an ordinary judge commit restoring R; it must report the decision record changed.
    origin: lens:code-correctness
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/driftengine/paths.go:69
    summary: The new findings path repeats the logs directory literal in violation of PATTERN-code-conventions.
    fix: Declare the logs directory segment once as a named constant in paths.go and use it in both LogsDir and Findings initializers.
    origin: lens:code-affected-set
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/driftengine/haltfindings.go:63
    summary: Group endpoint values follow judgement order instead of the reported row range.
    fix: Select first and last cells by the minimum and maximum rowOf values in each group, or sort the group by row before selecting endpoints. Extend TestDriftVerify with a valid shuffled judgement and assert the range and its corresponding endpoint values.
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/loomcli/driftwiring.go:292
    summary: No test exercises the production replacement-history selection and path filtering.
    fix: Exercise ReplacedDecisionRecord through the wired drift dependency in loomcli tests with multiple matching replacement commits, a newer trailer for another record, and no matching trailer. Assert the returned bytes and found value, including the snapshot-relative restriction required by F1.
    origin: lens:code-tests
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/driftengine/verify_test.go:811
    summary: The grouping test does not pin separation of otherwise identical cells with different rationales.
    fix: Add a row to the existing TestDriftVerify findings table with the same case, key and kind but different reasons, and assert that each reason retains a separate group and the correct endpoints.
    origin: lens:code-tests
  - id: F6
    severity: MEDIUM
    class: design
    location: internal/driftengine/verify_test.go:894
    summary: Counting bullets leaves the full findings file content contract untested.
    fix: Assert the expected full findings file contents in the existing TestDriftVerify table, including cell identities, old and new values, and rationales for cells omitted from the capped halt text.
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: design
    location: internal/driftengine/haltfindings.go:97
    summary: Separators and truncation suffixes are added outside the findings length budget.
    fix: Budget the complete emitted findings text, including separators, ellipsis and omission suffix, within haltFindingsLimit. Extend the existing public-surface test with an exact-boundary pair and an oversized multibyte rationale, asserting the emitted bound and valid UTF-8.
    origin: lens:code-correctness
  - id: F8
    severity: LOW
    class: design
    location: internal/driftengine/verify.go:754
    summary: The new before-session modification-time attribution has no reaching test.
    fix: Extend the existing TestDriftVerify attribution case with content that differs from the snapshot but has an mtime before the session opening, and assert the before-session wording.
    origin: lens:code-tests
---

### [BLOCKING] A historical replacement bypasses the guard

**Location:** internal/driftengine/judgesnapshot.go:383

**Issue:** A legitimate replacement writes R, a later parent addition produces R+A, and the snapshot captures R+A.
If the judge deletes A and commits R without a replacement trailer, the snapshot comparison rejects the deletion.
The fallback then compares against the historical R and exempts it.
`replacedDecisionRecord` selects the newest matching trailer without relating it to the snapshot's records HEAD.
This loses the guard's protection for decisions present when the judge started.

### [BLOCKING] The findings path duplicates its directory name

**Location:** internal/driftengine/paths.go:69

**Issue:** The new Findings initializer repeats the logs segment already used by LogsDir on the preceding line.
`pattern/PATTERN-code-conventions.md` requires one named constant for a name or path that can go stale.
Independent renames can separate the findings file from the directory the guard exempts.
The round's constraints require a BLOCKING finding for this violation.

### [MEDIUM] Shuffled rulings give incorrect endpoint values

**Location:** internal/driftengine/haltfindings.go:63

**Issue:** For rows arriving in order 3, 1, 2, the group reports range 1–3 but takes values from rows 3 and 2.
`ParseJudgement` validates membership and uniqueness without enforcing order.
`storeRulings` collects nonmatching cells in that input order.
The reported first and last values therefore need not belong to the range's first and last rows.

### [MEDIUM] Replacement selection is untested through its production wiring

**Location:** internal/loomcli/driftwiring.go:292

**Issue:** The snapshot fixture supplies replacement bytes directly.
`TestDriftDeps_CarryTheDecisionRecord` asserts only the decision path.
`TestDriftDeps_RunWhereTheStepsDirResolves` invokes capture and compare, not replacement lookup.
No test pins the new selection among replacement trailers or its record-path filter.

### [MEDIUM] Rationale separation is not pinned

**Location:** internal/driftengine/verify_test.go:811

**Issue:** The new rows use one repeated rationale or different cases.
They still pass if reason is removed from the grouping key.
Grouping by rationale is an explicit requirement of issue #147.

### [MEDIUM] Full findings are checked only by their count

**Location:** internal/driftengine/verify_test.go:894

**Issue:** The file assertion counts bullet prefixes.
Empty or duplicated bullets could satisfy it while losing the cells hidden by the halt cap.
The full file is the requested recovery source for those omitted details.

### [LOW] The cap omits formatting overhead

**Location:** internal/driftengine/haltfindings.go:97

**Issue:** Two findings of 750 ASCII characters pass the budget and then join into 1502 characters.
A truncated first finding adds an ellipsis after consuming the limit, and the omission suffix is also unbudgeted.
The emitted findings therefore exceed the stated cap.
The tests lens's oversized-finding coverage gap is consolidated here.

### [LOW] Before-session attribution is untested

**Location:** internal/driftengine/verify.go:754

**Issue:** The new test covers timestamps inside the window and after the halt.
`AChangeMadeBeforeTheSpawnIsNoJudgeEdit` snapshots the earlier content and emits no guard finding.
It cannot catch a regression in the new before-session attribution branch.

## Validation

Reviewed the committed range `72c3e192e7b9afff71558513de71d385538ff8cf..HEAD` against the decision record and live board body.
All four lens reports are saved beside this report.
No unrequested scope, duplicate added test, or loosened existing assertion was identified.
The focused tier0 gate could not acquire a slot because its holder record under `_bolt/.lym/gate` was read-only.
No test ran in that attempt; this environmental limit is not a code finding.
