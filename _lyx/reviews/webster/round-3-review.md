---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/commentlint/lint_test.go:79
    summary: the table-driven subtests of TestFindNewWrappedBreaks, TestCommandFor and TestTreeSitterLinkFindings do not call t.Parallel and no comment names a global state
  - id: F2
    severity: LOW
    class: scope
    location: internal/impactset/guards_test.go:28
    summary: no TestFindGuardTests row covers a test file that does not parse, the first of the three GuardScanError causes the guard scan documents
  - id: F3
    severity: LOW
    class: scope
    location: internal/commentlint/lint_test.go:72
    summary: no row covers a wrapped paragraph beside an indented code paragraph, though rows cover one beside a heading and one beside a list item
---

Range: `_lyx/loom/rework/` does not exist, so no live rework round applies and the range is `git diff $(git merge-base main HEAD)..HEAD`, with `main` read from `product.parent` in `_lyx/shed/test-tiers/status.json`.

Checked and found sound:

- Every Go comment line the diff adds or changes, scanned with a script over the diff's added line numbers (not the lint), pairing each added line with its neighbour in the same `//` block whatever the paragraph kind.
  Every break it flagged is a pair the base already had, so `PATTERN-comment-line-breaks` calls it carried over and not a finding (`internal/loomshed/gates.go` header and `hasBlockingFinding`, `internal/planglyph/repo.go` header, `internal/websterengine/beginbatch.go` re-resolve comment).
  No added line holds two sentences.
  `lyx loom lint-comments --base <merge-base>` over the branch passes.
- `isProseParagraph` in `internal/commentlint/lint.go` judges each paragraph on its own: `paragraphs` splits a block at blank comment lines, and only a paragraph holding indented code, a list item or a heading is skipped.
  Rows cover a wrapped paragraph beside a heading and beside a list item.
- `internal/impactset/guards.go` returns a `*GuardScanError` naming `file:line` for a marker in a `tmux` file and in an `llm` file, and `TestFindGuardTests` asserts `p_test.go:5` for both rows.
  `NewVerifyGate` turns that error into findings naming the file and line and the way forward, as card 10 says, and `TestVerifyGate_RoundScenario` reaches it through a misplaced marker.
- The markdown the diff adds follows semantic line breaks; the only unbroken long lines are list items and YAML in a code block.
- Build, `go vet` under `integration`, `tmux` and `llm`, and `go test` over `commentlint`, `impactset`, `loomshed`, `verifytree`, `loomcli`, `landingshed` and `cmd/lyx` pass, with the `integration` tier for the first five of those.
  The budget rows in `cmd/lyx/testdata/test-budget.yaml` equal the counted top-level tests.
- Scanning tests outside the marked set read only their own package directory, so card 9 leaves them unmarked.

### [LOW] Table-driven subtests do not call t.Parallel

**Location:** `internal/commentlint/lint_test.go:79`, `internal/impactset/derive_test.go:119`, `cmd/lyx/treesitterlink_test.go:161`
**Issue:** each top-level test is parallel, but its `t.Run` subtests are not.
`scribe:golang-testing` asks for `t.Parallel()` first in every test and subtest unless a comment names the state that prevents it, and `PATTERN-test-speed` asks the same of a test that touches no process-global state.
These three subtest bodies only read their table row and an immutable fixture, so nothing prevents it.
The sequential integration scenarios (`TestLint_OverARepository`, `TestDerive_OverARepository`, `TestLintCommentsCmd_OverARepository`, `TestVerifyGate_RoundScenario`, `TestVerify_PerCommandRecord`) are not part of this finding: each comment names the shared repository its ordered steps need.
**Fix:** add `t.Parallel()` as the first statement of each of the three subtest functions.

### [LOW] No guard-scan row for a test file that does not parse

**Location:** `internal/impactset/guards_test.go:28`
**Issue:** `GuardScanError` documents three causes: a file that does not parse, a marker not directly above a top-level test, and a marker in a file no `integration`-tier run compiles.
The table covers the last two; the parse failure in `guardNamesInFile`, which `guardScanFindings` tells the author to fix with "make every test file parse", has no row.
**Fix:** add a row whose file is not valid Go and whose `wantErr` is `impactset: parse dir/p/p_test.go`.

### [LOW] No row for a wrapped paragraph beside a code paragraph

**Location:** `internal/commentlint/lint_test.go:72`
**Issue:** the focus question is whether a block exempts only the paragraph that holds the code, list item or heading.
The rows show it for a heading and for a list item, but the indented-code branch of `isProseParagraph` is reached only by a block that is one paragraph, so a regression that exempted the whole block for indented code alone would pass.
**Fix:** add a row `a wrapped paragraph beside an indented code paragraph is flagged`, with the source `//\tcode`, `//`, `// a wrapped`, `// line.` and the flagged line the wrapped paragraph's first.
