# Plan verbs: glyphless test files, bare card N rewrites

Two faults in the loom plan verbs, both seen in task `warp-visibility`.

Closes #4
Closes #5

## What changes

### `edit-test-file` admits a test file with no symbols (#4)

A `_test.go` file that pins symbols only through blank-identifier package-level vars has no member glyph a card can name.
`lym quarry toc` lists no symbol for it, and `edit-test-file` refused the file path as an `Edit` or `Extends` target anyway.
The check now exempts a file the toc lists with no symbol, so a card can target the file itself.

- `planparser.CheckEditTestFile` takes a `glyphless func(path string) bool` and asks it only about a path it would otherwise refuse.
- `planglyph` supplies the test as `glyphlessFile`, which reads quarry's toc of the file whole.
  A file counts as glyphless only when the toc entry has no error, is not lossy, and has an empty symbol list.
  A file quarry cannot list or read, such as one that does not exist, is not glyphless and is still refused.
- The `edit-test-file` entry in `loom-plan-checks-spec.md` and the `**Edit:**` text in the plan-cards stencil describe the exemption.

### `insert-card` and `remove-card` rewrite bare card references (#5)

The renumber used to rewrite only card ids (`<N>-<slug>`) and report bare mentions in `prose_mentions`, leaving the planner to renumber them by hand.
It now rewrites a bare reference naming a moved card (`card N`, `cards N, M and K`, `cards N–M`) in the overview and every card file, and lists each rewrite under `rewritten_refs`.

- Every reference is read against the numbering before the call, so a number that already means the new numbering is not shifted twice.
- A reference with no faithful rewrite is left as is and its line is reported under `prose_mentions`, which now holds only those lines.
  That covers a reference naming the removed card, and a range the inserted or removed card falls inside, since the range would gain or lose a card.
- A card's heading line and the overview's Card Index lines are skipped.
- A match glued to a preceding hyphen, as in `insert-card 3`, is a command word and is left alone.
- A card id naming a moved card is still rewritten and listed under `rewritten_refs`; one naming the removed card is left as is and not listed under `prose_mentions`.

The old `proseMentions` scan after the write is gone; the mentions are now collected during the rewrite, before any file is touched, so the journal and restore path are unchanged.

`loom-plan-spec.md`, the `loomcli` package doc and the `insert-card` and `remove-card` help text describe the new behaviour.

## Design decisions

- The #4 fix exempts the file instead of making quarry report blank-identifier vars as members. Quarry's toc stays as it is, and the exemption holds for any glyphless test file, not only the blank-var case.
- #5 rewrites instead of only reporting, the first option the issue names, and keeps the report for the cases a rewrite would change the meaning.
- The rewrite of a bare range is all-or-nothing per reference: a range is either mapped on every moved number or left whole and reported.

## To check by hand

- Run `lym loom insert-card` and `lym loom remove-card` on a real plan whose prose uses `card N`, `cards N, M and K` and `cards N–M`, and confirm the rewrites and the reported lines read right. Only the unit tests exercised this.
- Run `lym loom validate-plan` on a plan whose card edits a test file holding only blank-identifier vars, with a live quarry toc, and confirm `edit-test-file` stays quiet. `TestIndex_AppendsPlanChecks` covers it only on a small temporary repository.
