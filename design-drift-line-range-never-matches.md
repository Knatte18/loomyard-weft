# drift: line-range expectation entries never match in diff mode lines

`drift-line-range-never-matches` · Note · bug, drift

Imported from [issue #141](https://github.com/Knatte18/loomyard/issues/141).

## Seen

In a darn run with drift scope full and `diff: lines` (key_patterns set), Drift-Expect wrote `expected.yaml` entries with `lines: - range: "1-"` or `"7-433"` per case.
The drift gate then failed on every changed value in those cases: the checker reports each changed value at line 0, so no line range ever covers it, even `"1-"`.

## Workaround

The parent amends the expectation with `lym drift amend <slug> --entries <file>` using key-based entries (`keys: - key: ..., n: ..., why: ...`), which match.

## Expected

A changed value carries its real line number so line-range entries match, or Drift-Expect is told to write key-based entries when the diff reports by key.
