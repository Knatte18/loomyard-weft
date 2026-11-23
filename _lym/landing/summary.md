# godocreflow lays comments out by the comment lint's own rules

`tools/godocreflow` was supposed to clear the wrapped-break findings of `lym loom lint-comments`, but it made more of them.
Three defects caused this:

- It skipped a whole comment group when any of its lines was a heading, list item or indented code, so wrapped prose next to them stayed wrapped.
- It split at every `, and`/`, or` followed by two words.
  The lint rejects such a break when the conjunction opens no clause (`, and returns an error`).
- Its last-resort wrap packed words greedily up to `-max-width` (default 100, indentation included).
  That broke lines mid-phrase, and the lint flags each such break.

## What changed

- `internal/commentlint` exports the rules the lint already applies, unchanged in behavior:
  `IsGenerated`, `IsDirective`, `IsProseLine`, `IsSemanticBreak` and `TextReachesLimit(raw, limit)`.
  The lint's own checks now call them, and the package doc no longer says the tool keeps a copy of them.
- `godocreflow` uses those rules instead of its own regexes:
  - Blank lines split a group into paragraphs.
    A paragraph with code, a heading or a list item stays as written, and each prose paragraph is laid out anew.
  - A break the tool makes survives only if `IsSemanticBreak` accepts it or the paragraph already broke between the same two words.
    A break that fails both tests is joined back.
    The lint treats a break that was already there as carried over, so it is not a finding.
  - Written breaks the lint accepts, such as one after a colon, are kept instead of being rejoined.
  - A line reaching `commentlint.LineLengthLimit` (text after `//`, indentation not counted) is split only at an accepted or written break.
    The tool takes the latest such break that keeps each piece under the limit.
    A line with no such break stays whole, and the run prints its file line for a hand rewrite.
  - The greedy wrap and the `-max-width` flag are removed.
  - The tool now honors every directive form the lint knows (`//lym:`, `//testtiming:`), not only `//go:` and `//nolint`.
    Before, it would join a `//lym:seam` line into the prose above it.

## Evidence

I ran the new tool over `internal`, `cmd` and `tools` of this branch and then ran `lym loom lint-comments`.
That changed 1623 files, with **0 wrapped-break findings** and 141 line-length findings.
The run named 291 lines that still reach 120 characters.
Each line-length finding I sampled was a line that was already over the limit and that the tool shortened without getting it under 120.
For example, `Told-geometry tier: the package takes …` became `Told-geometry tier:` plus a 141-character remainder.
The lint counts such a line as changed, so it flags it.
I discarded the tree changes after the measurement.
The issue reports the old tool's result as 624 findings rising to 916; I did not rerun the old tool here.

## Tests

`tools/godocreflow/reflow_test.go` gains rows for each defect:

- prose beside a heading or list is reflowed;
- a compound predicate is joined onto one line;
- a written break at a colon is kept;
- a line reaching the limit is split at a colon;
- a sentence over the limit keeps its written break;
- a line with no accepted break stays whole;
- a `//lym:` directive group is left alone.

The rows for the greedy wrap (`TestWidthFallbackAppliesToOverwideAtomicLine`, `TestWrapLongLine` and its rows in `TestReflowSource_LeavesUntouched`) are removed along with the wrap itself.
The existing `commentlint` tests cover the refactor of the lint's checks onto the exported functions.

## Design decisions

- The tool imports `internal/commentlint` rather than copying its rules, so its output and the lint cannot drift apart.
- A line still at or over the limit after splitting is left long, not wrapped by column.
  Only a rewrite can shorten it without a wrapped break, and that takes a person.

## Merge danger

**Door:** two-way
**Blast radius:** lint
