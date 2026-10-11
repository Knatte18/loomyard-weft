# Burler review/fix split with per-perch budgets

All four batches (cards 1 to 10) recorded `status: done`.

## Batch 01-review-carries-fix (cards 1 to 3, head `9e1f791e7`)

Every review finding carries its `fix`, and the current round's review gate checks it strictly.
`burlerengine` gained a review half and a fix half, each one session, with the lean fixer prompt and new stencils.
`shedadapters` gained a review row producer and a fix row producer, with the round's acceptance record.
Deviations were informational: scope warnings for helper symbols in the new files.

## Batch 04-split-burler-rows (cards 4 and 5, head `66b610f0c`)

Both recipes split every Burler row into a review row and a fix row, wired end to end.
`lyx burler run` runs the reviewer, then the fixer only when the review reached done.
Deviations were comment, doc and test updates for the new row names, plus the removed compile-time assertions in `shedadapters/burler.go` and its test, and edits to `loomrecipe/darnrecipe.go` and `burlercli/wiring.go`.

## Batch 06-row-name-prose (cards 6 to 9, head `314ad550e`)

Comments, specs and background files name the new rows.
A Bouncer told `AutoAccept` settles a spent budget as an accept.
`loom.yaml` gained per-perch budget and auto-accept keys, and each perch's rows carry its own budget and its Bouncer its flag.
Deviations were test files, `shedadapters/circling.go`, `shedrecipe/recipe.go` and `contracts/recipes/loom-recipe.yaml`.
The fork reported one `sed -i` and two heredoc appends against the editing rule; the audit raised no finding.

## Batch 10-retire-burler-round (card 10, head `b72553f9a`)

Deleted the one-row Burler round producer, the two-seat engine round, its stencils, the ready marker with `internal/burlermarker`, and the old row-name constants.
`roundComplete` now also requires the acceptance record.
Deviations included `loomengine/parentreview.go` and its test, which pointed at a deleted stencil, and stale-name comment fixes across several packages.
The fork reported the card gate's `./internal/burlerengine_test` path as nonexistent; every real package gate was green.
