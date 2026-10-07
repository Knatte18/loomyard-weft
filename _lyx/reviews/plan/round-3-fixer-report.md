# Round 3 fixer report — plan

All four findings were worked in the plan directory only, with no git commands and the overview's `approved:` key left at `false`.
After the edits `lyx loom validate-plan` reports `ok: true` with informational `create-new-unit` findings only.
No round-1 or round-2 fix regressed; I read those files only after saving the review.

## Changes

- **F1.** `internal/shuttleengine/attach_asking_test.go` joins card 5's Edit targets, and the card's known-tests list names `TestAttach_AskingReentryStartsAtRecordedOffset`: the output file exists before `Attach`, so the post-offset Stop finalizes Done, and the `AskingOffset` assertion goes.
  The search rule now covers every `_test.go` file of the package and makes each file the search hits an Edit target, so the file list follows the search and not an enumeration.
  This was a repeat of round 2's F1 in a narrower form, so the fix targets the method, not only the one file.
  Card 7 says it replaces card 5's adaptation of that test.
- **F2.** Card 13's driver-stencil bullet now has the generic line quote the tail's literal text ("nothing more than the tail \"Answer briefly, then continue your task.\""), the one place the stencil spells it out, and the other two replacement sentences refer back to it.
  The `claims_test.go` claim pins the literal.
- **F3.** Partly done.
  The `clock` to `Clock` rename stays an `Edit`, because I tried the `Rename` group the review proposed and the gate refused it: `rename-mechanic-missing`, and `containment-file-overlap` against the `wait.go` targets of cards 2, 5 and 6.
  The canonical `## Rename mechanic` is also written for file moves, not an in-file symbol rename.
  Card 14's Intent now states the rename is an AST-aware identifier rename followed by a grep for the old type name, and says why it is typed `Edit`.
- **F4.** Card 14's Coverage sentence now says the tests built through `withClock` exercise the setter, and that tests assigning `runner.clock` directly stay as they are.

## Deferred

- **F3, the `Clock` handle.**
  Card 15's `Uses:` still names only the `SetClock` handle, not `Clock`.
  A handle for an existing symbol is expressible only through a `Rename` pair, which the plan gate refuses here (see F3 above), and card 15 depends on card 14 already through `SetClock`.
  A format change that lets a rename of a symbol in a file other cards edit carry a handle is an operator decision, not something a plan fix can do.
