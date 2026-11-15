# Dobby clears a finished run's records so its slug is free again

A batten run that finished, through Land and Worktree-Teardown, left its records directory `_lym/shed/<slug>/` in the prime with state `done`.
That record refused every later start under the slug, so a second `lym beam merge-parent` task filed under a completed merge-in's slug could not start, and only a hand `lym batten cleanup --apply` freed it.
Now the dobby of the run's parent worktree removes a done run's records and scratch itself.
It does so once the run's landing is counted for the board tidy and its landing notice is decided.

Closes #128

## Design decisions

- The dobby clears the run, not batten's own last row.
  A batten pass cannot remove its own status file while it still reads it for its envelope,
  and the dobby's landing notice and board-tidy count read a run from its status file.
  Clearing in the batten pass could remove the run before the dobby saw it past Land, losing a member's landed line or the tidy count.
- The clear is a new step of the dobby tick, `clearDoneRuns` in `internal/dobbyengine/donerun.go`.
  It runs after the landing count and the landing notices, only while the tick schedules.
  It clears a listed done run only when:
  - no pass of it is in flight and its driver lock is free;
  - the landing-notice memory holds nothing of it;
  - and a landing it holds is counted.
  A cleared run logs `run <slug>: done run cleared` and leaves `CountedLandings`, so a later run under the slug counts its own landing.
  A failed clear is logged once per episode per slug and never fails the tick.
- `dobbyengine.Deps` gains a `ClearDoneRun` seam; `beamcli`'s dobby wiring fills it with `battencli.ClearDoneRun` over the prime.
  A nil seam clears nothing.
- `battencli.ClearDoneRun` is the recipe-agnostic sibling of `ClearMotifRun`.
  Both now go through one `clearDoneRun` with a recipe filter, under the slug's run lock and status lock,
  and they share the records commit, push spawn and write-back of the records on a failed commit.
  The write-back skips the child-recipe record when the run had none.
- The task branch is left to the pair removal, as before.
  Batten's Worktree-Teardown already deletes the task branch, locally and on origin, through the landed-work gate.
  A branch that gate keeps is kept on purpose, since its work is not proven landed, and `lym beam merge-parent` already skips a slug a leftover branch holds.
- The `lym batten run` and `lym batten cleanup` help texts and the `dobbyengine` package doc describe the clear.
  The done-slug refusal stays for a record the dobby has not cleared yet.

## To check

- `TestClearDoneRuns` in `internal/dobbyengine/donerun_test.go` pins each condition that holds a clear and the counted-landing drop.
- `TestClearDoneRun_RemovesADoneRunTheRecipeFilterAdmits` in `internal/battencli/cleanup_test.go` now covers the any-recipe filter beside the motif one.
- Bound: while a board-tidy agent is in flight the dobby counts no landing, so a run that landed meanwhile stays until the agent is settled.
