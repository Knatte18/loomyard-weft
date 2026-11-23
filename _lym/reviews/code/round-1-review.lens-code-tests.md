```yaml
lens: code-tests
outcome: reported
```

Task: review the diff's tests under the code-tests lens, listing each added or changed behaviour with the test row that pins it.

**Per behaviour, the test that pins it**

- `commentlint` exports `IsGenerated`, `IsDirective`, `IsProseLine`, `IsSemanticBreak` and `TextReachesLimit(raw, limit)`, and the lint's checks now call them.
  The existing `TestFindNewWrappedBreaks`, `TestFindNewLongLines` and `TestFindFileFindingsOrdersByLine` in `internal/commentlint/lint_test.go` pin this: rows for directives, code, list and heading paragraphs, generated files, the length-exempt rows, and lines at and under the limit.
- A prose paragraph beside a heading is reflowed: `TestReflowSource_ReflowsDocComments/prose paragraph beside a heading`.
- A prose paragraph beside a list is reflowed: `.../prose paragraph beside a list`.
- A heading, list or code paragraph stays as written: the same two rows, plus `TestReflowSource_LeavesUntouched/indented code example`, `/doc comment heading` and `/doc comment list`.
- A clause split the lint rejects is joined back: `.../compound predicate joined onto one line`.
- A written break the lint accepts is kept: `.../written break at a colon kept`.
- A one-line comment that reaches the limit is reflowed and split at an accepted break: `.../line reaching the limit split at a colon`.
- When the first usable break already gives a piece over the limit, the tool takes it anyway (the "earliest when none does" branch): `LeavesUntouched/sentence reaching the limit keeps its written break`, worked by hand: the 9-word piece is 45 characters against a limit of 40, so `cut` becomes 9.
- A line with no usable break stays whole: `LeavesUntouched/line reaching the limit with no break point stays whole`.
- A group holding a `//lym:` directive stays as written: `LeavesUntouched/lym directive`.
- A one-line comment under the limit stays as written: `LeavesUntouched/single-line comment` and `/single physical line group`.
- Generated files are skipped through `commentlint.IsGenerated`: `LeavesUntouched/generated file`.
- At each split the tool takes the **latest** break that keeps the piece under the limit: **none**.
- `planEdit` records in `edit.longLines` which reflowed lines still reach the limit: **none**.
- `reportLongLines` prints file:line for each such line, shifting line numbers by earlier edits, and returns the count: **none**. The reach report lists it as reached by no test.
- `main` uses `commentlint.LineLengthLimit`, the `-max-width` flag is gone, and the closing hand-rewrite line is printed: none. `main` is wiring and stderr text only a human reads, so this is no finding.

**Findings**

1. MEDIUM · design · `tools/godocreflow/main.go:131` (`reportLongLines`) and `tools/godocreflow/reflow.go:153-158` (`longLines`).
   Summary: no test pins which reflowed lines are reported as still over the limit, or the reflowed-file line numbers printed for them.
   Why it matters: the change description (`_lym/landing/summary.md:26`) and the package doc (`main.go:13`) say the run names each such line for a hand rewrite. That is output a person acts on, and the line-shift arithmetic (`main.go:141`) is easy to get wrong.
   Evidence: the reach report says "no mapped test reaches tools/godocreflow.reportLongLines (new)". No test reads `edit.longLines`; the tests compare only the applied source, `reflow_test.go:593` and `:747`.
   Fix: add a row that asserts `collectEdits`' `longLines` indexes for a group with one line left over the limit. Add a `reportLongLines` case with two edits, the first of which adds lines, asserting the returned count and the shifted line number.

2. LOW · design · `tools/godocreflow/split.go:326-339` (`splitLongLine`).
   Summary: no row tells "latest break that keeps the piece under the limit" apart from "first accepted break".
   Why it matters: the doc comment at `split.go:311` makes "latest" the contract.
   Evidence: `ReflowsDocComments/line reaching the limit split at a colon` (`reflow_test.go:578-586`) has exactly one candidate break. The `LeavesUntouched` rows at limit 40 have one written break or none. A mutant that adds a `break` after `cut = n` at `split.go:338` would pass every row.
   Fix: add a `ReflowsDocComments` row whose over-limit line has two accepted breaks, both early enough to keep the first piece under the limit, and expect the split at the later one.

Tool calls: none; the inherited context held everything this lens needed.
