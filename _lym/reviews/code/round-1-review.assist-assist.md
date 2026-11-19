---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/landingshed/finalize.go:755
    summary: Parent records checkpoints can hide an empty code landing in a one-repository hub.
    fix: "Make the landing emptiness comparison exclude records through fabric's records-path ownership, using RecordsPathsAt for the compared revisions rather than a hard-coded directory. Wire that code-only comparison into Finalize without changing unrelated ChangedPaths consumers. Add a regression case where the parent commits records after baseline capture and the task changes no code; EmptyDiff must remain true. Correct the landing summary's claim to match the implementation."
    origin: lens:code-affected-set
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/landingshed/finalize_test.go:630
    summary: The new empty-landing test runs serially despite independent fixtures.
    fix: Add t.Parallel after tiergate.Require in TestFinalize_RecordsWhetherTheLandedDiffIsEmpty and at the start of each independent subtest.
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize.go:755
    summary: A retry after a nonempty landing overwrites its record as an empty landing.
    fix: "Preserve the original landing result across retries after the parent-side merge. When the parent already holds this run's work, retain its existing landing classification instead of recomputing it against the current parent head; if that prior classification is unavailable, use the documented not-empty fallback. Extend the transport-push retry test to land a code change, fail the first push, retry Finalize, and assert the recorded EmptyDiff remains false. Also preserve a recorded true classification across the equivalent empty-landing retry."
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/darnengine/spec.go:133
    summary: Report-fragment read and fill failures have no regression coverage.
    fix: Add rows to TestDarnSpec_Refusals that enable ReportDir and respectively remove darn-fragment-report or give it an unresolved marker. Assert DarnTable returns an error and no seats in each case.
    origin: lens:code-tests
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/dobbyengine/landnotice.go:318
    summary: No test checks the beam follow-up on the new empty-landing return path.
    fix: Add a TestLandNotices row for a landedEmpty member of a beam with a pull request, expecting emptyLandingLine plus memberFollowUp and the existing notification clearing behavior.
    origin: lens:code-tests
---

### [BLOCKING] Parent records conceal an empty code landing

**Location:** internal/landingshed/finalize.go:755

**Issue:** The baseline precedes parent-record commits, but the comparison includes those records.
`mergeParent` captures the parent head before `landUnderLock` calls `CommitParentRecords`.
`parentRecordsCommitter` in `internal/loomcli/landingdeps.go:832` commits the parent's run records.
In a one-repository hub these are on the code branch.
`Fabric.ChangedPaths` in `internal/fabricengine/parenttips.go:119` supplies no exclusions to `diffNames`.
That helper in `internal/fabricengine/configchanges.go:141` runs an unrestricted name-only diff.
An empty code landing therefore reads as nonempty when only the parent's records changed after baseline capture.
Restoring receiving-side records during merge does not undo that difference against the earlier baseline.
This violates `PATTERN-single-repo-hub`, which requires code identity to strip records paths.
It also contradicts the landing summary's claim that records cannot make the landing count as nonempty.
The correctness lens independently found the same defect; it is consolidated here.

### [BLOCKING] Independent test rows run serially

**Location:** internal/landingshed/finalize_test.go:630

**Issue:** Neither the new test nor its rows call `t.Parallel`.
The file header names no process-global dependency.
`newTestDeps` allocates separate temporary directories and lock stores for each row.
`PATTERN-test-speed` requires parallel tests unless they touch process-global state.
The supplied constraints make violations of that rule blocking.

### [MEDIUM] A landing retry changes the recorded outcome

**Location:** internal/landingshed/finalize.go:755

**Issue:** A first call lands code from parent P to landing L and records `EmptyDiff: false`.
If the parent push fails, Finalize can be called again with L already on the local parent.
`mergeAndGate` calls `mergeParent` again, and `keepParentHead` now retains L as the baseline.
The parent-side merge is a no-op, but `recordLanding` still compares L to L and overwrites the flag with true.
Dobby then reports that no file changed although this run landed code.
`TestFinalize_TransportPushFailure_ReturnsClassifiedError` already exercises this supported retry shape.
Its re-step row checks ordering but not the new classification.

### [MEDIUM] Required report errors are untested

**Location:** internal/darnengine/spec.go:133

**Issue:** The new report fragment can fail at its read or fill boundary.
Every existing refusal row leaves `ReportDir` empty, so those paths are bypassed.
Silently dropping the required report directive on either error would leave the current assertions green.
This is a coverage gap in prompt construction, not a request to assert static stencil prose.

### [MEDIUM] Empty beam-member notice lacks follow-up coverage

**Location:** internal/dobbyengine/landnotice.go:318

**Issue:** The new empty-landing row passes an empty follow-up.
Existing beam-member rows exercise other return paths.
Removing the follow-up from this new return would therefore pass those tests.
The summary explicitly promises that the beam-member follow-up remains at the end of the line.

## Other focus conclusions

The committed report instruction and docs profile setting trace to Goal 1.
Flagging instead of refusing traces to Goal 2; no unrequested Land refusal was added.
Missing and old one-line landed records, trailing newlines, and flag propagation have direct coverage.
An unavailable baseline or failed path read returns not-empty in the immediate calculation.
The retry and records-checkpoint cases above defeat the broader claim about the baseline.
The report fragment is engine-neutral and uses the engine-aware stencil reader.
`LoadConfig` performs lexical locality validation and normalizes the directory.
The default report path is formed from that directory and the task slug.

The reported mutation at `finalize.go:741` changes the warning condition after the callback.
It does not remove the already-asserted empty-diff argument.
The mutation at `landed.go:43` changes the file mode, not the serialized marker.
Neither survivor establishes the missing flag or marker assertion suggested by the focus question.

Validation performed: `go build ./internal/landingshed ./internal/darnengine ./internal/shedrun` passed.
No test gate was rerun; the available shell tool cannot set the required 3600000 ms command timeout.
The supplied gate reports were used as review evidence, not treated as a new test run.
