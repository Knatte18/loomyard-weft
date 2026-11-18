---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/shedrecipe/entries_burler.go:68
    summary: The staged replacement patch includes working-tree and untracked changes outside the committed review range
    fix: Supply staging with a committed base-to-HEAD patch through the fabric boundary, using the review rubric's base. Keep the existing CodeSubject working-tree patch for Bouncer identity checks. Add a regression case with a committed change, an uncommitted reversal of that change and an untracked file; the staged patch must retain the committed change and exclude the untracked file.
    origin: lens:code-affected-set
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/burlerengine/template_test.go:93
    summary: The contained template tests lack the required keep annotations for their distinct assertions
    fix: Add a testtiming:keep directive directly above TestTemplate_FillsWithAllMarkers naming its missing-marker error assertions, and another directly above TestTemplate_OptionalDirectives naming its empty-marker output and placement assertions. Preserve both tests.
    origin: lens:code-tests
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/burlerengine/diffpieces.go:73
    summary: Flattening a valid nested package path into one filename can prevent the review from starting
    fix: Bound the filename component produced by pieceFileName independently of the package path. Use the sequence with a bounded readable suffix or digest, and retain the full package path in diffPiece.Package and the rendered list. Add a row with several valid directory components whose combined length exceeds the filesystem component limit.
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: internal/shedadapters/burlerreview_test.go:434
    summary: Diff-reader timing and refresh behavior are not covered by the new producer rows
    fix: Extend the existing profile, retry and probe/resume behavior tables with a counting reader that returns changing patches. Assert no construction-time read, one read shared by retry attempts, no read on handled probe/resume exits, and a fresh patch on a later spawning call.
    origin: lens:code-tests
  - id: F5
    severity: MEDIUM
    class: design
    location: internal/burlerengine/diffpieces_test.go:27
    summary: Staging tests omit write failures, assistant delivery and the promised boundary preference
    fix: Extend TestWriteDiffPieces with a blocked directory and a conflicting piece output path, asserting the returned errors, and with exact expected pieces for hunk-first and line-end splitting. Extend an existing assisted-round test with a non-empty Diff and assert that both explore files name the same existing pieces. Keep these cases in the existing behavior tests.
    origin: lens:code-tests
  - id: F6
    severity: LOW
    class: design
    location: internal/burlerengine/diffpieces.go:105
    summary: Header parsing can assign a changed file to the wrong package
    fix: Parse the destination filename using Git's quoted-path escaping and header rules instead of deleting quotes and taking the last space-b-slash occurrence. Add grouping rows for quoted paths and a destination containing a literal space-b-slash sequence, including an unrelated package with the misleading suffix.
    origin: lens:code-correctness
---

### [BLOCKING] Stage the committed review subject

**Location:** internal/shedrecipe/entries_burler.go:68

**Issue:** `subjectDiff` copies the Bouncer's `Patch` into the review profile.
`codeSubject` obtains it from `ReadTaskPatch` in `internal/loomcli/subjects.go:169`.
`internal/fabricengine/taskpatch.go:84` compares the base against the working tree;
its untracked-file loop at line 92 appends additional changes.
This is intentional Bouncer behavior, covered by `TestFabric_ReadTaskPatch`.
The receiving rubric instead specifies `git diff <base>..HEAD`.
The new diff-pieces stencil tells the reviewer to replace that command with the staged pieces.
An uncommitted reversal can therefore hide a committed defect, and an untracked file becomes review subject.
The correctness and affected-set lenses identified the same defect; this finding consolidates them.

### [BLOCKING] Record why the contained tests remain

**Location:** internal/burlerengine/template_test.go:93

**Issue:** The supplied redundancy report lists both template tests as contained by `TestAnswerParentAsk`.
The latter asserts parent-ask state and report contents, not marker errors or marker placement.
The template tests therefore remain useful, but neither has its required keep directive.
Both the rubric and `PATTERN-test-speed` require that annotation for a retained contained test.
The directive makes violations of listed constraints BLOCKING.

### [MEDIUM] Bound the output filename

**Location:** internal/burlerengine/diffpieces.go:73

**Issue:** Replacing slashes with hyphens puts the entire package path into one filename component.
Three directory components of 100 characters each are valid individually but produce a component over 300 bytes.
This filesystem reports a 255-byte component limit.
The resulting `WriteFile` error propagates through `Engine.Review` before a reviewer starts.
The sequence prefix already provides uniqueness; preserving an unbounded path in the filename is unnecessary.

### [MEDIUM] Pin the diff reader's lifecycle

**Location:** internal/shedadapters/burlerreview_test.go:434

**Issue:** The new rows each perform one spawning call with a constant-returning reader.
They check the value delivered, but cannot detect an early read, repeated retry reads or a stale later-round patch.
Existing retry and probe/resume tables leave the reader nil.
The placement after live-review handling and before the attempt loop is new behavior without an asserting row.

### [MEDIUM] Complete the staging behavior coverage

**Location:** internal/burlerengine/diffpieces_test.go:27

**Issue:** All splitter rows use writable fresh directories and expect success.
Neither filesystem failure branch is exercised.
The oversized-file rows assert only size, reconstructed contents and multiple pieces;
arbitrary byte slicing would satisfy them despite losing the promised hunk and line boundaries.
The engine staging test starts no assistant, and existing assistant tests supply no diff.
Dropping the assistant's staged-piece list would consequently leave those assertions passing.
This consolidates the tests lens's staging-failure, assistant-delivery and boundary-preference findings.

### [LOW] Parse Git filenames before grouping

**Location:** internal/burlerengine/diffpieces.go:105

**Issue:** Removing quotes does not decode Git's escaped pathname representation.
For the header `diff --git a/src b/pkg/x.go b/src b/pkg/x.go`, the last ` b/` is inside the new pathname.
The parser reports `pkg` instead of `src b/pkg` and can group the file with an unrelated directory.
The diff bytes survive, but the requested per-package partition and its labels do not.

Review notes:

- Reviewed `71d6ee1a044d76998b1a3ff37a5eaec9113b83d5..HEAD` and its commit messages against the Sketch and board entry T6.
- The byte cap holds through package, file, hunk, line and oversized-line splits.
  Binary and rename-only sections retain their bytes; empty input produces no pieces.
- Sequence prefixes distinguish similar flattened package names, and write errors are propagated.
- Unset staging, an absent subject and failed reads follow their documented paths.
  The producer reads after live-review handling and once before its attempt loop.
- The optional prompt block disappears without pieces, and assistant profile copying retains the piece paths.
- Tapestry, darn and sampler enable staging only on their Code review rows.
  No implementation group exceeds the board entry's requested scope.
- The tier-0 gate could not acquire a slot because its hub gate directory was read-only in this sandbox.
  No tests ran; the attempted gate result is an execution limitation, not a code finding.
