---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: tools/godocreflow/split.go:263
    summary: The final join recreates an over-limit line from input lines that all fit.
    fix: In layoutParagraph, resolve rejected breaks without discarding the fitting written boundaries restored by length splitting. Recheck the final layout against the length rule and retain the original fitting layout when no valid replacement fits. Add the reproduced multi-line subject case to the existing reflow test table with literal expected source.
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: tools/godocreflow/reflow_test.go:201
    summary: No test distinguishes the new latest-fitting-break policy from choosing the first eligible break.
    fix: Add rows to TestReflowSource_ReflowsDocComments with several eligible written or semantic boundaries and literal expected output. Cover choosing the latest fitting boundary and choosing the earliest boundary when no prefix fits, through source reflow rather than helper tests.
  - id: F3
    severity: BLOCKING
    class: design
    origin: lens:code-tests
    location: tools/godocreflow/main.go:131
    summary: The new unresolved-line count and adjusted source locations have no behavioral coverage.
    fix: Cover reporting through the formatter command behavior with multiple comment groups, including an earlier edit that changes the line count. Assert the unresolved count and final file locations, including unchanged long groups, without pinning cosmetic diagnostic wording.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: tools/godocreflow/reflow.go:150
    summary: Unchanged unsplittable long comments disappear from the promised hand-rewrite diagnostics.
    fix: Collect unresolved eligible comment locations independently of replacement edits in collectEdits and planEdit, and report them in main even when no group changes. Preserve correct line shifts from preceding edits without counting or writing unchanged files as edits.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: handler
    location: tools/godocreflow/reflow_test.go#TestReflowSource_LeavesUntouched
    summary: Width assertions were removed without the measured Loosened-Check trailer the rubric requires.
    fix: Add a Loosened-Check trailer on a commit in the review range naming the removed width guarantees in TestWidthFallbackAppliesToOverwideAtomicLine and TestWrapLongLine and a measured before-and-after cause for replacing them. Retain the independent word-preservation assertion in the existing reflow behavior test if that contract remains required.
---

### [BLOCKING] The final join defeats length splitting

**Location:** tools/godocreflow/split.go:263

**Issue:** The final join can undo length splitting and create a fresh length violation.
This contradicts the board goal to respect the line-length limit.
The following input has fitting comment lines:

```go
package p
// Foo reads every record from the store and writes a detailed summary of the results, and the
// collection of data from the current transaction and the prior transaction
// is validated against the complete collection before the final result can be returned.
func Foo() {}
```

Running `go run ./tools/godocreflow -check /dev/stdin` with that input joins the first two comment lines.
It reports `/dev/stdin:2: line still reaches 120 characters`.
Length splitting preserves the written break before `is validated`.
That removes the finite verb from the next line considered by the comma-break check,
and the final join then merges the subject fragment into the preceding line without another length pass.
The original written break after `and the` would avoid the new violation.
Both the correctness lens and the handler reproduced this with check mode; no target file was changed.

### [BLOCKING] Break selection lacks a distinguishing assertion

**Location:** tools/godocreflow/reflow_test.go:201

**Issue:** The new policy chooses the latest eligible fitting cut, or the earliest cut when none fits.
The added colon case and preserved-written-break case each offer only one eligible cut.
An implementation that always selects the first cut would satisfy both.
This is missing coverage of added behavior under the test-economy rubric and `PATTERN-test-speed`.

### [BLOCKING] Diagnostic data is untested

**Location:** tools/godocreflow/main.go:131

**Issue:** The new reporting path calculates final line numbers across edits and counts unresolved lines.
The existing source-reflow tests discard edit metadata and never invoke reporting.
Deleting the unresolved indexes or the line-shift adjustment would leave those assertions unchanged.
The supplied reach report also identifies `reportLongLines` as unreached.
This concerns computed diagnostic data, not prose wording, under `PATTERN-test-speed` and the test-economy rubric.

### [MEDIUM] Unchanged long lines receive no diagnostic

**Location:** tools/godocreflow/reflow.go:150

**Issue:** The equality return occurs before unresolved lines are collected.
The command then skips files with no edits before invoking reporting.
An eligible unsplittable long line therefore produces only `0 file(s) would change`.
The handler confirmed that outcome with this input:

```go
package p

// Foo reads every record from the configured store and validates every field before returning the complete collection to the caller.
func Foo() {}
```

The new command documentation and commit description promise to name such lines for hand rewriting.
The affected-set lens independently reported the same defect; it is consolidated here once.

### [MEDIUM] Removed checks lack the required measured explanation

**Location:** tools/godocreflow/reflow_test.go#TestReflowSource_LeavesUntouched

**Issue:** The diff deletes the width-bound assertion and word-preservation assertion from the old fallback test.
It also deletes the exact wrapping expectations in `TestWrapLongLine`.
Both commits carry `Root-Cause` trailers but neither carries `Loosened-Check`.
Removing greedy wrapping is within scope;
the rubric nevertheless requires a measured trailer for assertions removed from tests present at the base.
The commit history does not supply that required explanation.
