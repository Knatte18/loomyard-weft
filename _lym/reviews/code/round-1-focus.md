---
round: 1
focus:
  - "internal/commentlint/linelength.go: check the counted text, limit, and exemptions against scribe:prose 'Line breaks' as installed (120 breaks, 119 passes; after `//`, without indentation or list marker; headings, blockquotes and table rows exempt; a URL, code span or identifier that alone passes the limit exempt). Does any rule the skill words differ in the code, such as trailing whitespace, a marker form listItemPattern misses, or a code span or word measured differently from the skill?"
  - "internal/commentlint/linelength.go findNewLongLines: the 'changed' test is exact trimmed literal text found anywhere in the base's comment blocks. Does a moved or duplicated long line, or a line that matches an unrelated base line, wrongly escape or wrongly trigger? Does it match the board entry's 'lines the diff adds or changes'?"
  - "internal/commentlint/lint.go findFileFindings and Lint: the generated-file guard moved out of findNewWrappedBreaks. Is every caller of findNewWrappedBreaks still guarded, and do end-of-line comments and directive comments stay unchecked by the new rule as doc.go says?"
  - "Finding.Rule and Finding.String: every consumer of Finding (internal/loomcli/lintcomments.go, internal/gatecli/failurereport.go, internal/loomshed/verifygate.go, internal/gateslot/tiers.go) renders the new form and nothing is left that prints or parses the old file:line: text form. Does the change to each consumer stay within what a line-length finding failing the gate like a break finding needs?"
  - "internal/commentlint/doc.go and docs/overview.md: do the stated rules match the code, and is the unchecked part of the skill's rule (where to split) stated as the board entry asks? Do the docs the repository's own constraints name for this kind of change land in the diff?"
  - "internal/commentlint/lint_test.go and the two updated integration tests: is each new behavior covered once, and does any new test duplicate an existing one or assert only prose?"
  - "_lym/landing/summary.md: check each claim (exemptions, carry-over by exact text, rendered form in every report, tests named) against the diff."
---

Seed focus for round 1. The decision record holds only the board entry under `## Goal`, so the amended entry is the measuring stick.
