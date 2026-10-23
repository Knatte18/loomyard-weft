---
verdict: APPROVED
---

The amendment under review is commit `de55116e`, the reed pane-listing separator.
The orchengine compact-hold change is settled and was not re-reviewed.
All four forks returned without findings, and the handler's own review found none.

## Verdicts on the focus questions

- **Every site on the tab.**
  `paneTailSeparator` is a tab in `parse.go`, and both `-F` formats embed it: `paneListFormat` in `overlay.go` and `windowPaneFormat` in `ownwindow.go`.
  Both parsers of the tail go through `splitPaneTail`: `parsePaneList` and `parseWindowPanes`.
  No `\x1f` or `\037` is left in a tmux-listing context anywhere in the repository.
  The remaining `\x1f` uses are `fabricengine`'s git `--format` separators and are unrelated.
- **Scope.**
  The decision record's "Added after Discussion (parent, 2026-10-23)" entry licenses the switch in `parse.go`, the `-F` formats in `overlay.go` and `ownwindow.go`, the parse comment and the `doc.go` format line.
  The diff touches those files, the matching test listings, and one more `doc.go` word: "whitespace-separated" became "space-separated", which is accurate because `parsePaneList` splits on a space.
  It is its own commit, as the entry asks.
- **A tab in a pane title.**
  A probe on tmux 3.4 showed `select-pane -T` rejecting a tab and an OSC 2 title having it stripped, so a tab never reaches the title field.
  A tab in the start command stays whole, because `SplitN` keeps the remainder in the last part.
  The change description's bound, that a tab in a title would split the tail, is a conservative hypothetical that cannot occur on this tmux, and is not a defect.
- **Test economy.**
  The test edits swap the separator in existing listings and add no test.
  The tab split is pinned by the converted rows of `TestParsePaneList` and `TestParseWindowPanes`, and the listing users in `lifecycle_test.go` and `strand_test.go` follow the constant through `faketmux_test.go`.
  The hermetic tests cannot see a mismatch between a format constant and the parser, because the fake builds its output from the same constant.
  The `go:build tmux` integration tests are the pin for that, and the verify gate decides them.
- **Change description claims.**
  The tmux 3.4 escaping claim holds: a probe showed `\x1f` arriving as the literal text `\037` and a format tab passing through verbatim.
  The tier 2 run was not repeated here, since it is the verify gate's to decide.

## Rejected

- None: no fork reported a finding.
