---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D3 (walk read error), Scope, Constraints
    summary: D3 adds a begin-batch and recover-batch refusal on a guard-walk read error but never lands its refusal-spec row, and PATTERN-refusal-way-forward is absent from Constraints
    fix: >-
      In D3, after the sentence ending "rerunning the verb once the path reads is the way forward", add "The refusal lands as a transient row in the webster section of contracts/specs/refusal-spec.md, either its own row or the grouped raw I/O row widened to name a read of the worktree's guard walk, and a test reaches it." Add contracts/specs/refusal-spec.md to Scope's docs-in-the-same-commit bullet. Add a Constraints bullet "PATTERN-refusal-way-forward: D3's walk read-error refusal names its way forward, lands its refusal-spec row in the same commit, and a test reaches it."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 Bound, Open risks last bullet
    summary: D4's Bound and Open risks rely on the batch's review to catch a wrong pin loosening, but no per-batch review exists
    fix: >-
      In D4's Bound, replace "Such an edit stays visible as a scope-guard deviation in the batch record and the batch's review." with "Such an edit stays visible as a scope-guard deviation in the batch digest's deviations list, which Master's run summary names per batch, and in the committed range Webster-Review reviews." In Open risks, replace "only the batch's review of the scope-guard deviation catches it" with "only Webster-Review of the committed range, with the summary's per-batch deviation list, catches it".
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D2, drop clause parenthetical
    summary: The drop clause's condition says every Go file, but its parenthetical says file-level old sides covering all its files, which never holds for a package directory that also holds a non-Go file
    fix: >-
      In D2's drop paragraph, change "or file-level ones covering all its files" to "or file-level ones covering all its Go files".
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md D2 directory walk, Acceptance criteria
    summary: The walk claims to skip what go list skips but omits vendor directories, and the acceptance rows cover only the testdata skip, not the underscore and dot prefixes or the nested go.mod
    fix: >-
      In D2, change "skipping `testdata` directories, directories whose names begin with `_` or `.`, as `go list` does," to "skipping `testdata` and `vendor` directories and directories whose names begin with `_` or `.`, as `go list ./...` does,". In Acceptance criteria, replace "A `_test.go` file under a `testdata` directory is never scanned." with "A `_test.go` file under a `testdata` or `vendor` directory, a directory whose name begins with `_` or `.`, or a directory under a nested `go.mod` is never scanned."
---

### [BLOCKING] D3's walk read-error refusal lands without its refusal-spec row

**Location:** `_lyx/discussion/decision-record.md` D3, Scope, Constraints

**Issue:** D3 makes begin-batch and recover-batch refuse on a read error in the guard walk, naming the path, with "rerunning the verb once the path reads" as the way forward.
That is a new refusal in webster, a module `PATTERN-refusal-way-forward` binds: it must land its row in `contracts/specs/refusal-spec.md` in the same commit and a test must reach it.
The webster section's grouped raw I/O row covers "`stat`, `mkdir`, `read` or `write` of a run file", and a walk over the worktree's source directories is not a run file, so no existing row covers it.
The record's Scope enumerates the docs and contracts the change updates, `loom-plan-spec.md` among them, and omits `refusal-spec.md`;
its Constraints name five PATTERN entries and omit this one.
`Plan-Write` reads only the decision record, so the omission steers the plan toward a pattern violation.
The acceptance row "A read error in the guard walk refuses begin-batch, naming the path" supplies the test half; the spec row is missing.

### [MEDIUM] D4's visibility bound cites a per-batch review that does not exist

**Location:** `_lyx/discussion/decision-record.md` D4 Bound; Open risks, last bullet

**Issue:** D4's Bound says a pin edit "stays visible as a scope-guard deviation in the batch record and the batch's review", and Open risks says "only the batch's review of the scope-guard deviation catches it".
Webster has no review per batch.
Record-batch writes the scope guard's files into the digest's `deviations` (`internal/websterengine/recordbatch.go`, `digest.go`), which is informational only;
Master's summary names each batch's deviations (`contracts/stencils/webster/webster-template-master.md`, the `summary_path` paragraph);
and Webster-Review reviews the committed range (`internal/websterengine/doc.go`), with no rubric line about deviations.
So the bound names the wrong backstop.
The real one is weaker: a reviewer of the whole range who must spot a loosened pin in the diff.
D4's bound on steering (fix the code first, move the pin only when the pin must move) and on scope (only a file the guard failure names) are stated and hold.

### [MEDIUM] D2's drop condition says every Go file in one place and all files in its parenthetical

**Location:** `_lyx/discussion/decision-record.md` D2, the drop paragraph

**Issue:** The drop paragraph's main clause drops a directory "when a card at or before it renames out or deletes every Go file of that directory", and its parenthetical restates that as "file-level ones covering all its files".
A package directory often holds non-Go files too, such as an embedded stencil or a `.md` file.
Read literally, the parenthetical never fires for such a directory: it stays in the guard step after its last Go file has left, and `lyx gate test` on a directory with no Go file fails, which is gap 2 again.
The main clause, the closing "a directory that still holds Go files at that card stays in the step", the Bound and the acceptance row "renames out or deletes every Go file" all say Go files, so the parenthetical is the outlier.

On the rest of the focus for D2's drop: the drop applies only from the emptying card on, the acceptance rows pin both the card before it and a one-file move, and the Bound states what it skips.
Its claim that a renamed guard test runs in the batch gate holds: `gateScopeOf` keeps a target directory when the target names Go source (`td.NamesGo`), so a new directory that does not exist yet at begin-batch still reaches the batch gate.

### [LOW] The guard walk's skip list is not what `go list ./...` skips, and two of its skips have no acceptance row

**Location:** `_lyx/discussion/decision-record.md` D2, the directory-walk sentence; Acceptance criteria

**Issue:** D2 says the walk skips `testdata`, `_`- and `.`-prefixed directories "as `go list` does", plus anything under a nested `go.mod`.
`go list ./...` also skips `vendor` directories, which the walk does not name, so a hub with a vendored tree would have its vendored `_test.go` files scanned, and a misplaced marker there would turn into a `GuardScanError` that drops the step.
This repo has no `vendor` directory, so it does not bite here.
The only acceptance row is "A `_test.go` file under a `testdata` directory is never scanned";
the prefix skip and the nested-module skip have none, though this worktree has both cases: `_lyx` and `.lyx` at the root, and `internal/planglyph/testdata/glyphchain/go.mod`.

## Focus verdicts

On the D3 focus item, the verdict is no defect beyond F1.
D3's Bound states that the scan error recurs at every later begin-batch until the file is fixed, and that guard coverage may be absent for the rest of the run.
The walk read error has its own way forward, a re-run, and the record keeps it apart from the `GuardScanError` rationale.
Open risks covers the persistent `GuardScanError`.
A read error is a refusal, not a silent degrade, so it needs no Open risks entry.
