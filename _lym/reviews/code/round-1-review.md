---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/landingshed/finalize.go:751
    summary: In a one-repository hub, the parent records commit Step 2b makes after parentHeadMerged is read counts as a changed path, so a landing that carries no code is recorded as not empty and dobby's flag never fires.
    fix: "In landedDiffEmpty, judge emptiness on code paths only: drop every path under the records paths fabric's RecordsPathsAt names (through a fabricengine helper or a records-excluding Deps seam wired in internal/loomcli/landingdeps.go, never a hard-coded `_lym` literal) before testing len(paths). Do not move the base past Step 5b's GitHub merge, which would make every pull-request landing read as empty. Add a row to TestFinalize_RecordsWhetherTheLandedDiffIsEmpty whose ChangedPaths(parent0, landed1) returns only a records path and want true. Update the internal/landingshed/doc.go paragraph and the summary's \"a run's records alone never make a landing count as non-empty\" claim to match."
    origin: lens:code-correctness
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/landingshed/finalize_test.go#TestFinalize_RecordsWhetherTheLandedDiffIsEmpty
    summary: The new test and its subtests never call t.Parallel and name no process-global state, against PATTERN-test-speed.
    fix: Add t.Parallel() right after tiergate.Require(t, 0) in TestFinalize_RecordsWhetherTheLandedDiffIsEmpty and as the first line of its t.Run body; newTestDeps gives each row its own temp dirs, so nothing is shared.
    origin: assist:assist
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/landingshed/finalize.go:751
    summary: A Finalize re-step after a failed parent push re-reads parentHeadMerged as the landing commit already on the local parent, so ChangedPaths(L, L) is empty and a landing that carried code is overwritten as empty-diff.
    fix: "Keep the first landing's classification across a re-step: when the landed-commit record already names the same landing commit, keep its EmptyDiff instead of recomputing; with no such record, record not empty as the doc's fallback says. One way is to have recordLanding skip the flag computation when the parent already held the task's work at the pass's start and the landed-commit record exists. Extend the re-step subtest of TestFinalize_TransportPushFailure_ReturnsClassifiedError, or add a row, asserting the flag recorded on the re-step stays false after a first pass that landed a code change."
    origin: assist:assist
  - id: F4
    severity: LOW
    class: design
    location: internal/dobbyengine/landnotice.go:318
    summary: No test row pins that a beam member's follow-up ends the new empty-landing line, which the change description claims.
    fix: "Add a TestLandNotices row for a beam member whose run is landedEmpty, built like the existing member rows, wanting emptyLandingLine + memberFollowUp."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: design
    location: internal/darnengine/config.go:139
    summary: LoadConfig accepts a report_dir under a records tree such as _lym or .lym, where a report is stripped at the merge or never tracked, so it never reaches the parent branch.
    fix: In the report_dir check, also refuse a cleaned value equal to or under a records root named by internal/lymdirs (the tracked records dir and the dot-lym dir), with an error naming the key, the value and the same way forward; add a TestLoadConfig row for one such value.
    origin: lens:code-correctness
  - id: F6
    severity: LOW
    class: consistency
    location: internal/darnengine/spec_test.go:184
    summary: The new TestDarnSpec row asserts static stencil prose, the report heading and the make-change fragment's closing sentence in reportAfterStep4, plus the phrases around the filled paths.
    fix: Replace reportHeading and reportAfterStep4 with assertions on the filled values alone, `docs/research/` and `docs/research/fix-the-thing.md`, and on their position after the Step 4 heading; in the no-report row assert the absence of the filled report path instead of the heading.
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: design
    location: internal/darnengine/spec.go:133
    summary: DarnTable's new report-passage error path, a missing or unfillable darn-fragment-report with report_dir set, has no test row.
    fix: Add a TestDarnSpec_Refusals row that sets Config.ReportDir and removes darn-fragment-report from the stencils dir, asserting DarnTable returns an error.
    origin: assist:assist
---

### [BLOCKING] Parent records commit hides an empty landing

**Location:** internal/landingshed/finalize.go:751

**Issue:** `mergeParent` keeps `parentHeadMerged` (finalize.go:1215).
Afterwards, Step 2b's `commitParentRecords` (finalize.go:381) commits sibling runs' status records on the parent branch, before the parent-side merge.
`Fabric.ChangedPaths` runs an unrestricted `git diff --name-only` (`internal/fabricengine/configchanges.go:141`).
In a one-repository hub those records are tracked on the code branch, so they appear in the path list.
A landing that carried no code then reads as not empty, and dobby still says `landed as <commit>`: the very case the board entry asks to flag.
PATTERN-single-repo-hub says code identity strips the records paths.
The change description's claim that a run's records never make a landing count as non-empty covers only the task's own records, which the merge restores; it misses the parent's records commit.

### [BLOCKING] New Finalize test runs serially

**Location:** internal/landingshed/finalize_test.go#TestFinalize_RecordsWhetherTheLandedDiffIsEmpty

**Issue:** PATTERN-test-speed requires `t.Parallel` unless a test touches process-global state, named in a comment at the test or atop its file.
The new test has neither, and `newTestDeps` gives each row its own fixtures.
Sibling tests in the file that run serially are existing debt.
This test is new.

### [MEDIUM] A re-step after a failed push records a code landing as empty

**Location:** internal/landingshed/finalize.go:751

**Issue:** The first pass lands L and records `EmptyDiff: false`, then the parent push fails with a transient error.
The re-step runs `mergeAndGate` again, which is the supported path `TestFinalize_TransportPushFailure_ReturnsClassifiedError` re-steps.
`keepParentHead` then reads L, which already holds the task's work, and the parent-side merge leaves the head at L.
`ChangedPaths(L, L)` is empty, so `RecordLanding` overwrites the record with `EmptyDiff: true`.
Dobby then says no file changed for a run whose work landed.

### [LOW] Beam follow-up on the empty-landing line is untested

**Location:** internal/dobbyengine/landnotice.go:318

**Issue:** The new return path appends `followUp`, but the only empty-landing row passes none.
The member rows use `landedLine` alone, so a version that dropped the follow-up from this path would pass.

### [LOW] report_dir may point into a records tree

**Location:** internal/darnengine/config.go:139

**Issue:** `filepath.IsLocal` is the only check, so `_lym/research` and `.lym/x` load.
A report there is restored away at the merge or never tracked.
It never reaches the parent branch, which is the key's stated purpose.

### [LOW] TestDarnSpec pins stencil wording

**Location:** internal/darnengine/spec_test.go:184

**Issue:** `reportHeading` and `reportAfterStep4` quote the new fragment's heading and the make-change fragment's last sentence.
The row also quotes the words around the filled paths.
These are new assertions on static stencil prose, which the rubric's test-economy bullet names.

### [LOW] Report passage error path untested

**Location:** internal/darnengine/spec.go:133

**Issue:** Every refusal row leaves `ReportDir` empty, so `reportPassage`'s read and fill errors are never reached.
A change that swallowed them and dropped the directive would pass.

## Rejected

- code-affected-set F1 (the darn Code-Review rubric may flag the report file as unlicensed scope): the board entry itself makes the report a required output of every docs run, and the passage has the change description name it. The reviewer-deletes-report chain is hypothetical, and its fix would edit a rubric the entry does not ask to change.
- Assistant F4's MEDIUM and F5's MEDIUM severities: lowered to LOW (F7 and F4 here). Each is a single untested branch of plain propagation or formatting.
- Assistant F5 is merged into F4, which code-tests F1 also raised.
- The mutation survivors are not findings. At finalize.go:741 the mutant negates the `err != nil` that only chooses whether to log, and the empty-diff argument is already asserted by `TestFinalize_RecordsWhetherTheLandedDiffIsEmpty`. At landed.go:43 the mutant changes the file mode, which no reader depends on.

## Other focus conclusions

- Scope: the diff takes the entry's "flag" branch and adds no refusal path in Land. `recordLanding` only logs. Every change traces to one of the two goal points or is forced by one.
- The landed-commit file: a missing file, an old one-line file and a trailing newline all read back correctly. The flag travels from `RecordLanding` through `WriteLandedCommit`, `ReadLandedCommit` and `DobbyRuns` to `RunView.EmptyLanding` and the dobby line.
- `report_dir`: `LoadConfig` refuses `..` and absolute paths. The docs profile template sets the key. The passage is engine-neutral, read through `stencilstore.ReadForEngine`, and fills `<report_dir>/<slug>.md` as the default path.
- The commit messages' and summary's dobby wording matches `landnotice.go:318`. The one summary claim the diff does not bear out is the one F1 names.
