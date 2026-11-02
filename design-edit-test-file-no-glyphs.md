# validate-plan edit-test-file rule cannot target a test file with no glyphs

`edit-test-file-no-glyphs` · Note · bug, loom, quarry

Imported from [issue #4](https://github.com/Knatte18/loomyard/issues/4).

## What happened

A plan card needed to edit a Go test file that pins symbols only through blank-identifier package-level vars.
`lym quarry toc` reports no symbols for such a file.
`lym loom` `validate-plan`'s `edit-test-file` rule refuses the file as an Edit target and asks for member glyphs, which the file does not have.
The card could not plan the edit, and the plan-review finding asking for it had to be deferred.

## Why it is abnormal

A file that exists and is a legitimate edit target becomes unplannable because the toc cannot produce a glyph for it.
The rule's demand cannot be satisfied by the planner.

## What to do

Let `edit-test-file` accept a file with no toc symbols (for example by targeting the file or a line range), or make `lym quarry toc` surface blank-identifier vars as targetable members.

Provenance: task `warp-visibility`, note `burler-plan-r1.md`.



_Moved from #909 in the repository before it was recreated._
