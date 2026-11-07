---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/commentlint/linelength.go:28
    summary: a heading of level two or deeper (`## Title`) is not exempt from the length rule, although doc.go says a heading is exempt
    fix: In internal/commentlint/linelength.go isExemptFromLength, test the text against a new pattern `^#{1,6}(\s|$)` instead of headingPattern, leaving headingPattern (lint.go:45) unchanged for the wrapped-break rule. Add a `// ## <120+ chars>` row to TestFindNewLongLines in lint_test.go expecting no finding.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/commentlint/linelength.go:47
    summary: the changed test matches a long line against any base comment line by exact text, so a long line the diff adds as a copy of a base line escapes
    fix: In findNewLongLines, build known as a map[string]int counting each trimmed literal in the base's comment blocks, and skip a new line only while its count is above zero, decrementing it on each match. Add TestFindNewLongLines rows for a long line duplicated in the new text (the second copy is flagged) and a long line moved within the file (not flagged).
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/commentlint/lint.go:84
    summary: no test feeds a file holding both a wrapped break and a long line, so the line ordering findFileFindings adds is unpinned
    fix: Add a test in internal/commentlint/lint_test.go that calls findFileFindings on a text with a long line before a wrapped break and the reverse, and asserts the findings come back ordered by line with both rules present.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gateslot/template.yaml:17
    summary: the shipped gate-config comment still describes the comments lint as fixed-column-wrapped breaks only
    fix: In internal/gateslot/template.yaml line 17, change "comments (fixed-column-wrapped breaks in // comments)" to name both rules, for example "comments (fixed-column-wrapped breaks and over-long lines in // comments)".
  - id: F5
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/websterengine/doc.go:119
    summary: the package doc still calls the lint "the comment line-break lint"
    fix: In internal/websterengine/doc.go line 119, change "the comment line-break lint" to "the comment line lint".
---

### [LOW] Heading exemption misses level-two and deeper headings

**Location:** internal/commentlint/linelength.go:28 (pattern at internal/commentlint/lint.go:45)
**Issue:** `headingPattern` is `^#(\s|$)`, so `// ## Title` followed by 120 characters is flagged. The skill exempts headings and `doc.go` states the exemption without a level limit. The test pins only `# `.

### [LOW] A copied long line escapes the changed test

**Location:** internal/commentlint/linelength.go:47
**Issue:** `known` is a flat set of every trimmed base comment line, so a new long line equal to any base comment line is skipped, including a second copy the diff adds. The board entry asks for lines the diff adds or changes. A line merely moved is unchanged and rightly passes.

### [LOW] Ordering of mixed findings is unpinned

**Location:** internal/commentlint/lint.go:84
**Issue:** `findFileFindings` sorts the two rules' findings by line, and no test gives one file both rule kinds, so removing the sort breaks nothing. This is a behavior the diff adds with no covering row.

### [LOW] Gate template comment names only the break rule

**Location:** internal/gateslot/template.yaml:17
**Issue:** The diff updates the `LintComments` doc in `tiers.go`, `docs/overview.md` and the verb help, but not this shipped template text, which still describes the lint as covering breaks only.

### [NIT] Webster package doc keeps the old lint name

**Location:** internal/websterengine/doc.go:119
**Issue:** It says "the comment line-break lint", which no longer matches the lint's scope or the wording the diff gives `docs/overview.md`.

## Rejected

- A line-length finding is not run end to end through the `lint-comments` verb or the gate report (lens code-tests, item 3): both render through `Finding.String`, which `TestFindingString` and the updated break-finding integration tests pin, so a second case would duplicate their coverage.
- The `loomshed` `commentLintFindings` message is not asserted (lens code-tests): the message is prose, and the `String` path is pinned elsewhere.
- A `+` list marker is counted (lens code-correctness note): the skill names no such marker form, and the lens did not report it.
- Counting trailing whitespace: the skill is silent, and gofmt strips it from comments.
