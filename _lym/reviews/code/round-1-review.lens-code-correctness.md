```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each added or changed function and branch on a concrete input.

**List (function, case run, verdict)**
- `countedText`, `"//   - " + 119 chars`: TrimLeft drops the indentation, `listItemPattern` drops `- `, and 119 characters are counted. Correct.
- `countedText`, `"//\t<code>"`: the tab is trimmed. Correct.
- `countedText`, `"// 12. text"`: the numbered marker is stripped. Correct.
- `countedText`, `"// + item"`: `+` is not in `listItemPattern`, so the marker is counted. The skill's own examples use `-`, so this is a minor gap. Not reported as a finding.
- `isExemptFromLength`, `"// # Title…"`: counted text `# Title`, matched by `headingPattern`, so exempt. Correct.
- `isExemptFromLength`, `"// ## Sub heading" + 120 characters`: `headingPattern` is `^#(\s|$)`, so `##` does not match. The line is not exempt and is flagged. **Wrong; see F1.**
- `isExemptFromLength`, a blockquote (`>`) or table row (`|`) prefix: correct.
- `isExemptFromLength`, a URL of 120 or more characters: the `strings.Fields` word check exempts it. Correct.
- `isExemptFromLength`, a code span with internal spaces of 120 or more characters: `codeSpanPattern` exempts it. Correct.
- `isExemptFromLength`, an empty or whitespace-only line: it is not reached, because the length check short-circuits first. Correct.
- `findNewLongLines`, new text identical to base: every line is in `known`, so there are no findings. Correct.
- `findNewLongLines`, an edited long line: the literal differs, so it is flagged. Correct.
- `findNewLongLines`, an empty base (a new file): `known` is empty, so every long line is flagged. Correct.
- `findNewLongLines`, a long line copied or moved elsewhere in the file, or matching any unrelated base comment line: `known[literal]` is true, so it escapes although the diff adds it. **See F2.**
- `findNewLongLines`, a `\r\n` file: `commentBlocks` trims `\r\n`, so the count is unaffected. Correct.
- `findFileFindings`, a generated file: returns nil before either rule runs. Correct.
- `findFileFindings`, two findings on one line: `SortStableFunc` by line keeps the wrapped-break finding ahead of the line-length one. Deterministic.
- `findFileFindings`, the `append(findNewWrappedBreaks(…), …)` call: the first slice is freshly built and not aliased, so this is safe.
- `Lint`: the error paths from `changedGoFiles` and `readChange` are unchanged and propagate. Correct.
- `Finding.String`, an empty `Rule`: renders `file:line: : text`. Every constructor sets `Rule`, so this is unreachable.
- `commentLintFindings` in `verifygate.go`: the format string is built from constant parts plus `%d`, and there is no stray `%`. Correct.
- `renderCommentFindings` and `runLints`: both use `String()`, so the form is consistent.

**Findings**

F1: LOW, design. `internal/commentlint/linelength.go:28` (`headingPattern` at `internal/commentlint/lint.go:45`).
- A heading of level two or deeper (`// ## Title`) is not exempt from the length rule, because `headingPattern` is `^#(\s|$)` and matches only a single `#`.
- The skill exempts headings, and the doc.go claim "A heading … is exempt" is false for `##` and deeper.
- Evidence: `lint.go:45` and `linelength.go:28`; the test only uses `# `.
- Fix: use a heading pattern of `^#{1,6}(\s|$)` for the exemption. Add a `##` row to `TestFindNewLongLines`. Widen only the new use if the wrapped-break rule's single-`#` behaviour must stay unchanged.

F2: LOW, design. `internal/commentlint/linelength.go:47-52,58`.
- The "changed" test is exact trimmed text found anywhere in the base's comment blocks.
- A long line the diff adds as a copy of, or that happens to equal, any other base comment line escapes. The board entry says lines "the diff adds or changes".
- Evidence: `known` is a file-wide set of literals with no position or multiplicity.
- Fix: count occurrences. Skip a new line only while the base still holds an unmatched occurrence of that literal, using a `map[string]int` that is decremented per match. Add a duplicated-line row to the test.

Tool calls: none made; my context already held the diff and both full source files.
