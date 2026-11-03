# Plan verbs: validate-plan cannot target a glyphless test file, insert-card leaves bare card N mentions stale

`plan-verbs-glyphless-and-refs` · Task · bug, loom, quarry · running · Code-Burler-Review

Two faults in the loom plan verbs, from [issue #4](https://github.com/Knatte18/loomyard/issues/4) and [issue #5](https://github.com/Knatte18/loomyard/issues/5), both seen in task `warp-visibility`.

## 1. `validate-plan` cannot target a test file with no glyphs (#4)

A plan card needed to edit a Go test file that pins symbols only through blank-identifier package-level vars.
`lym quarry toc` reports no symbols for such a file, and `validate-plan`'s `edit-test-file` rule refuses it as an Edit target and asks for member glyphs the file does not have.
The card could not plan the edit, and the plan-review finding asking for it had to be deferred.

Expected: `edit-test-file` accepts a file with no toc symbols, for example by targeting the file or a line range, or `lym quarry toc` surfaces blank-identifier vars as targetable members.

## 2. `insert-card` leaves bare "card N" mentions stale (#5)

Inserting a card renumbered every following card, but `insert-card` rewrote only the `<N>-<slug>` ids.
About twenty bare "card N" mentions in plan prose stayed at the old numbers and had to be renumbered by hand, with no warning from the verb.

Expected: `insert-card` also rewrites bare "card N" references, or reports each one it leaves unrewritten so the planner can fix them.
