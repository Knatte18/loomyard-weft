```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. List the tests that pin each behaviour the diff adds or changes.

**Per behaviour, the test that pins it, or none**

- **Limit boundary (119 passes, 120 flagged):** `TestFindNewLongLines` rows "one character under the limit" and "at the limit" (`lint_test.go` ~L150–151). Pinned.
- **Counted text excludes the space after `//`, indentation and the list marker:** rows "one character under the limit, the space after the slashes uncounted", "indentation and a list marker are not counted", "a list item at the limit is flagged" and "an indented code line at the limit". Pinned.
- **Characters counted, not bytes:** the `é` row. Pinned.
- **Carry-over by exact text, and an edit to a long line triggers:** rows "carried over unchanged" and "an edit changes". Pinned.
- **Carry-over matches a line anywhere in the base's comment blocks:** no row for a moved or duplicated line, or a base line in a different block. The summary says "stands unchanged in a comment block of the base", and `findNewLongLines` implements it with a flat set. No test fails if the set becomes per-block or per-position.
- **Exemptions (heading, blockquote, table row, URL or unbroken word, code span):** one row each. Pinned. The code-span row (span with spaces, 122 characters) exercises the span branch separately from the word branch.
- **Exemption by `strings.Fields` word:** only the URL row. Pinned.
- **Directive, end-of-line and generated files unchecked by the new rule:** rows "a directive line is not checked", "an end-of-line comment is not checked" and "a generated file is never flagged" in `TestFindNewLongLines`. Pinned.
- **Generated-file guard moved into `findFileFindings`:** the wrapped-break test's generated row (`lint_test.go:125`) now runs through `findFileFindings`, since `flaggedLines` calls it. Pinned for both rules.
- **`Finding.Rule` set to `RuleWrappedBreak` on break findings:** `flaggedLines` filters by rule, so the existing break rows fail if the tag is wrong. The two integration tests pin the rendered `wrapped break:` text. Pinned.
- **`Finding.String` form `file:line: rule: text`:** `TestFindingString`, plus the `loomcli` and `gatecli` integration tests. Pinned.
- **`findFileFindings` merges both rules and orders by line:** none. No row gives a file with both a wrapped break and a long line, so `slices.SortStableFunc` could be removed with every test still passing.
- **Line-length finding fails through `lint-comments` (exit 1, rendered text):** none. The `loomcli` integration test only covers a wrapped break. The shared `String` path and the unchanged `len(findings) > 0` branch carry it, so this is LOW at most.
- **Line-length finding fails the gate report, and the `lintWayForward` text:** none for a line-length finding in the `gatecli` integration test. The test builds the expectation from the `lintWayForward` map, so it does not pin the new wording. Same branch as above, so LOW at most.
- **`loomshed` `commentLintFindings` new message and `String` rendering:** no test asserts the new text. The `verifygate_integration_test.go:233` scenario was not updated, so it presumably asserts only the failure, not the message. Wording is prose and exempt, but the switch to `f.String()` is unpinned there. LOW.
- **`gateslot` `LintComments` doc, `overview.md` and `doc.go` text:** prose, exempt.

**Findings**

1. LOW, consistency, `internal/commentlint/lint.go:84-93` (`findFileFindings`): no test covers a file with both rule kinds, so the line ordering is unpinned. Fix: add one row or test that feeds a base and new text with a long line after a wrapped break (and the reverse) and asserts the order of the findings.
2. LOW, scope, `internal/commentlint/lint_test.go:~L150-167` (`TestFindNewLongLines`): the "known anywhere in the base" carry-over has no row. Fix: add rows for a long line moved within the file and for a long line identical to a base line in a different block.
3. LOW, scope, `internal/loomcli/lintcomments_integration_test.go:64` and `internal/gatecli/test_integration_test.go:1458`: only a wrapped-break finding is run end to end, so no line-length finding is shown to fail the verb or the gate. Fix: add a line-length case to each existing scenario. Do this only if the handler judges the shared path insufficient. These could instead be rejected as duplicate coverage of `String`.

**Tool calls**

1. `grep` of the test files for `Code generated`, `commentLintFindings` and the old wording, to find the generated-file rows and see whether any test asserts the `loomshed` message. The manifest lacked the existing test rows.
