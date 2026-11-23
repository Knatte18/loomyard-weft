---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: tools/godocreflow/split.go#layoutParagraph
    summary: The layout can leave a changed comment line that reaches the length limit, so a run adds line-length findings where the written paragraph had none.
    fix: "In layoutParagraph (tools/godocreflow/split.go), after the final joinRejectedBreaks, check every laid-out line with commentlint.TextReachesLimit. When a line reaches the limit and is not identical, word for word, to one of the paragraph's written lines, return the written lines unchanged (stripped of their one leading space, the form layoutParagraph returns) so the paragraph keeps its written layout. This covers both causes: splitLongLine's partial split of a legacy over-limit line, and the final join that rejoins a cut splitLongLine accepted against rest[n:] but the lint rejects against the shorter piece that follows. Add rows to TestReflowSource_ReflowsDocComments or TestReflowSource_LeavesUntouched in tools/godocreflow/reflow_test.go with literal sources: the three-line paragraph ending `results, and the` / `collection ... prior transaction` / `is validated ... returned.` at the default limit stays as written; and a written one-line paragraph over a small limit whose only accepted break leaves a piece still over the limit stays as written."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: tools/godocreflow/reflow.go#planEdit
    summary: The hand-rewrite report names lines the tool never reflowed and misses unsplittable long prose lines in groups that need no edit, breaking the package doc's promise that the run names each line left whole.
    fix: "Compute the lines to report apart from the replacement edits: in collectEdits, for each eligible group, record the source line of every prose-paragraph line (commentlint.IsProseLine) in the group's final text that reaches the limit, whether or not planEdit returns an edit, and skip list, heading and indented-code lines. Return those line numbers, in the numbering of the reflowed file, beside the edits, and have main in tools/godocreflow/main.go print and count them for every file, including a file with no edits, without counting such a file as changed. Drop edit.longLines and the line-shift loop in reportLongLines if the new return carries the numbers."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: tools/godocreflow/reflow_test.go
    summary: No test pins which lines the run reports for a hand rewrite or the reflowed-file line numbers it prints; the reach report lists reportLongLines as reached by no test.
    fix: "Add a test in tools/godocreflow/reflow_test.go that calls collectEdits (or the function F2's fix introduces to return the report's line numbers) on a source with two comment groups: an earlier group whose reflow adds lines, and a later group holding a prose line that still reaches a small limit. Assert the exact reported line numbers as literals, including the shift the earlier group causes, and assert that a long list-item line is not reported. Assert data, not the stderr wording."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: tools/godocreflow/reflow_test.go#TestReflowSource_ReflowsDocComments
    summary: No row tells the documented latest-fitting-break choice in splitLongLine apart from taking the first accepted break.
    fix: "Add a row to TestReflowSource_ReflowsDocComments whose one over-limit line holds two accepted breaks (for example a colon and a later semicolon), both early enough that the piece before each stays under the row's limit, and whose remainder after the later break also stays under it; expect the literal output split at the later break only."
  - id: F5
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: The Evidence section sets the new tool's result against the issue's 624-to-916 figures without a before figure from the base tool on the same tree.
    fix: "In _lym/landing/summary.md under ## Evidence, run the base commit's tools/godocreflow over the same internal, cmd and tools trees, run lym loom lint-comments, and give its wrapped-break and line-length counts beside the new tool's; or remove the comparison with the issue's figures and state the new counts alone. Update the line-length count after F1's fix."
---

### [BLOCKING] Layout leaves changed lines at or past the length limit

**Location:** tools/godocreflow/split.go#layoutParagraph

**Issue:** The board entry asks the tool to respect the line-length limit, because it produced more lint findings than it cleared.
The lint excuses only an unchanged over-limit line (`internal/commentlint/linelength.go`, `findNewLongLines`), so any changed line that reaches the limit is a new finding.
The layout creates such lines in two ways.

First, `splitLongLine` splits a written over-limit line even when no accepted or written break brings every piece under the limit: it cuts at the earliest break, or leaves a remainder over the limit.
The cut changes the line, so the lint flags the remainder, though the line left whole would have passed as unchanged.
The change description's own run shows the effect: 141 line-length findings, each a line already over the limit that the tool shortened without bringing it under 120 (`_lym/landing/summary.md`, Evidence).

Second, `splitLongLine` judges a comma-and-conjunction cut against all remaining words (`rest[n:]`), but the final `joinRejectedBreaks` judges it against the next piece alone.
When a later cut at a written break falls inside the clause, the next piece loses its verb, the final join rejects the break and rejoins the line, and no length pass follows.
Worked through by hand on this written paragraph, every line under 120 characters:

```go
// Foo reads every record from the store and writes a detailed summary of the results, and the
// collection of data from the current transaction and the prior transaction
// is validated against the complete collection before the final result can be returned.
```

`reflowText` splits at `results,` and the clause check passes (it reads `is`).
`splitLongLine` cuts the second piece at the written break before `is validated`.
The final join then sees `and the collection ... prior transaction`, which holds no verb candidate, and joins it onto the first line, giving a line of about 165 characters.
The assistant reproduced the same case with `-check`, which reported the joined line as still reaching 120 characters.
The correctness lens raised both causes separately; one fix in `layoutParagraph` removes both.

### [MEDIUM] The hand-rewrite report covers the wrong lines

**Location:** tools/godocreflow/reflow.go#planEdit

**Issue:** The package doc says a line with no break "stays whole, and the run names it for a hand rewrite".
`planEdit` returns nil when the reflow changes nothing, before it collects `longLines`, and `main` skips a file with no edits before it calls `reportLongLines`.
So an unsplittable long prose line in an otherwise settled group is never named.
At the same time `longLines` counts every line of the group's new text, so a long list-item or indented-code line that `reflowParagraph` returned untouched is reported as a reflowed line that still reaches the limit.
After F1's fix, a paragraph kept as written is exactly the case the report must name.

### [MEDIUM] The long-line report is untested

**Location:** tools/godocreflow/reflow_test.go

**Issue:** The reach report lists `reportLongLines` as reached by no test, and no test reads `edit.longLines`: both reflow tests compare only the applied source.
Which lines are reported and the line-shift arithmetic across earlier edits are behaviour the diff adds and a person acts on.
The stderr wording is not the subject; the line numbers are.

### [MEDIUM] Latest-fitting-break choice is untested

**Location:** tools/godocreflow/reflow_test.go#TestReflowSource_ReflowsDocComments

**Issue:** `splitLongLine`'s doc makes the latest break that keeps the piece under the limit the contract.
The row `line reaching the limit split at a colon` offers one candidate break, and the limit-40 rows in `TestReflowSource_LeavesUntouched` offer one written break or none.
A version that took the first accepted break would pass every row.

### [LOW] Evidence compares against figures from another run

**Location:** _lym/landing/summary.md

**Issue:** The Evidence section reports 0 wrapped-break and 141 line-length findings after the new tool, set beside the issue's 624-to-916 figures, and says the old tool was not rerun.
An improvement claim needs a before figure from the base code on the same tree.

## Rejected

- Assistant F5, removed width assertions need a `Loosened-Check:` trailer: the deleted tests pinned `wrapLongLine` and the width fallback, which the change removes as the board entry asks, so no retained behaviour lost its check; the word-preservation contract stays pinned by the exact-output rows of `TestReflowSource_ReflowsDocComments`.
- Assistant F2 and F3 at BLOCKING: kept as F4 and F3 at MEDIUM, since each is a coverage gap on behaviour whose correctness F1 and F2 already block or fix.
