---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

## Changes

- F1, F2, F3 and F4 share `tools/godocreflow/reflow.go` and `reflow_test.go`, so one commit holds them.
  - F1: `layoutParagraph` (`split.go`) returns nil when a laid-out line reaches the limit and is not a trimmed written line of the paragraph.
    `reflowParagraph` (`reflow.go`) then returns the written lines.
    This differs from the review's wording only in mechanism: nil replaces returning the stripped written lines, so a written line without a leading space is not given one.
    Rows added: `paragraph whose layout would join a line over the limit stays as written` (the review's three-line paragraph, limit 120) and `line whose only break leaves a piece over the limit stays whole` (limit 30) in `TestReflowSource_LeavesUntouched`.
  - F2: `planEdit` returns the group's long prose lines apart from the edit, `collectEdits` converts them to reflowed-file line numbers and returns them, and `main` prints and counts them for every file, including a file with no edits.
    `edit.longLines` and the shift loop in `reportLongLines` are gone.
  - F3: `TestCollectEdits_ReportsLongProseLinesInReflowedNumbering` asserts the line number `10` across an earlier group that grows by one line, and that a long list-item line is not reported.
  - F4: row `line reaching the limit split at the latest break that fits` in `TestReflowSource_ReflowsDocComments`, with two colon breaks.
- F5: `_lym/landing/summary.md` Evidence now sets the base tool against the new tool on the same tree, measured with `lym loom lint-comments` after each run, the tree discarded between runs.
  Base: 3593 files changed, 15748 wrapped-break findings, 0 line-length findings.
  New: 1589 files changed, 0 and 0, with 606 prose lines named for a hand rewrite.
  The summary's description of the layout, tests and design decisions was updated for F1 and F2.
  I did not commit `_lym/landing/summary.md`, since this round never runs git against `_lym`.
- Self-check follow-ups, in a second commit: the package doc in `main.go` and the `layoutParagraph` doc in `split.go` now state the kept-as-written rule; the summary's 606-line sentence no longer claims no break could shorten them.

- Gate repair, in a third commit: the doc comment of `TestCollectEdits_ReportsLongProseLinesInReflowedNumbering` wrapped a sentence across lines, a wrapped-break finding; it now has one sentence per line.

- Second gate pass: its only open items are that no mapped test reaches `main` or `reportLongLines`.
  I added no test and changed nothing: both only print the line numbers `collectEdits` returns, which `TestCollectEdits_ReportsLongProseLinesInReflowedNumbering` already pins as data, and a test of the stderr wording would pin text only a person reads.

`lym gate test ./tools/godocreflow` passed before the commits; the self-check edits changed comments only, and `go vet` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW, package doc in `main.go` lacked the kept-as-written rule: fixed.
- fix-consistency, LOW, summary's "no break can shorten" clause too loose: fixed.
- fix-consistency, LOW, `splitLongLine` doc "earliest when none does" can read against `layoutParagraph`'s doc: fixed differently; I removed the contradicting sentence in the `layoutParagraph` doc and left the `splitLongLine` doc, which is accurate for that function.
- fix-proposal, LOW, same overstated 606-line sentence: fixed (merged with the consistency finding above).

## Disputed

None.

## Deferred

None.

## Rulings

None.
