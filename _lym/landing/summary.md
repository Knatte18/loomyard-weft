# Stage the reviewer diff in pieces that fit one Read

A code reviewer used to build the task's diff itself.
In one run it wrote a diff of about 3,600 lines to a scratch file, read it in five spans, had its last Read refused for size, and retried with `allow_large`.
That session cost about three times the row median, with the whole diff in its context.

Now burler writes the committed diff out as one file per package, each small enough for a single Read, and lists the files in the reviewer's explore step.

## What changes

- `internal/burlerengine/diffpieces.go` splits a unified diff into pieces of one package each.
  A piece is capped at 40,000 bytes and 1,500 lines.
  A package over the cap splits at file boundaries into numbered parts.
  A single file over the cap splits at hunk starts, then at line ends, on rune boundaries.
  Each piece of a split file after the first repeats the file's header when that header fits within half a piece.
  Piece file names bound the package part at 80 bytes.
- The pieces are written beside the reviewer's instruction files.
  The new stencil `contracts/stencils/burler/burler-diff-pieces.md` lists them, and `burler-step-1-explore.md` gains an optional `diff_pieces` marker that renders nothing in a round with no diff.
  The text tells the reviewer to read the pieces instead of running `git diff` over the range, and to read every needed piece in one message.
  Where the rubric narrows or refuses the range (a merge from the parent, an unreadable base), the rubric's range governs and the pieces are only a reading aid.
- `BurlerReview` rows take an optional `stage_diff: true` key.
  The `darn`, `sampler` and `tapestry` code-review rows set it.
  A row that sets it while `Env.CodeDiff` is nil is refused when the recipe is built.
- The diff comes from a new `Fabric.ReadCommittedDiff` in `internal/fabricengine/taskpatch.go`: the commits from the merge base to HEAD, with no uncommitted edit and no untracked file.
  `internal/loomcli/subjects.go` wires it through `committedCodeDiff`, and shares the base lookup with `codeSubject` through `recordedTaskBase`.
  A standalone run, with no recorded parent branch, gets no diff.
- `BurlerReviewProducer` reads the diff once per spawning call.
  A failed read is logged as a warning and the reviewer runs without pieces, as before.

## Design decisions

- The diff is the committed range, not the patch `ReadTaskPatch` builds for the review's identity.
  The reviewer's rubric reads the commits, so an uncommitted edit would show it lines the review range does not hold.
- The stencil states that the rubric's range wins over the pieces, so the pieces cannot widen a range the rubric narrowed.
- The cap counts bytes and lines.
  Bytes alone let a file of very short lines pass the Read tool's default line limit.

## To check by hand

No end-to-end review run was done against a large branch.
Run a code review on a branch with a diff well over 3,000 lines and check that the reviewer reads the pieces and runs no whole-range `git diff`.

Findings a review left open, each to check in the diff:

- `piece-cap-bytes-only`: `diffPieceMaxBytes` and `diffPieceMaxLines` in `internal/burlerengine/diffpieces.go` bound a piece; check that the byte cap alone is safe for dense or binary patches.
- `split-order-and-assistant-pieces-unpinned`: check that piece order and the assistant reviewer's pieces are pinned by tests in `internal/burlerengine/diffpieces_test.go` and `assist_test.go`.
- `staged-patch-not-committed-range`: check that the diff staged is the committed range (`ReadCommittedDiff` in `internal/fabricengine/taskpatch.go`) and not the working-tree patch.
- `stencil-overrides-rubric-range`: check that `contracts/stencils/burler/burler-diff-pieces.md` leaves the rubric's range in force when it narrows or refuses the review range.
- `template-tests-lack-keep-markers`: check that `internal/burlerengine/template_test.go` and `contracts/stencils/markers_test.go` cover the `diff_pieces` marker and that the marker survives the template's keep-marker handling.
