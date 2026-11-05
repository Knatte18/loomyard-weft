# Motif runs: own profile recipe, discussed state, live board status

Three motif faults seen in hub Models-LYMHUB, beam `duplication-and-import-followups`.

## What changes

- **Own profile recipe.**
  The motif recipe no longer sets `profile_recipe: tapestry`, so a motif task takes the `motif` block of its run profile and the `motif` default like any other recipe.
  `internal/runprofile/template.yaml` gains a `motif` block (the two Discussion bounce keys) in every profile and a `motif: solid` default.
  The `profile_recipe` key is removed from `shedbuild`, `loomrecipe` and `shed-recipe-spec.md`, which nothing else used.
- **A finished motif reads as finished.**
  A recipe with no Finalize row, which lands nothing and ends at the Discussion approval, now leaves the new board status `discussed` (`boardengine.StatusDiscussed`) instead of `draft`.
  The board list, the `--text` listing and the README show it with a `discussed` marker, and the README gives it its own `Discussed` subsection ahead of `Draft`.
  The status releases the run lock like `draft` did, and `null` through `lym board set-status` releases it the same way.
- **Board status follows the run.**
  A batten transition at any row listed before Run-Shed (`battenrecipe.RowsBeforeRunShed`) now writes `<state> · <row>` on the task's board entry, so the status no longer sticks at Worktree-Create.
  After Run-Shed the inner run writes the status itself, which gives `running · Discussion-Write` for a motif.
  A failed board write only warns, and the next transition retries.
- **`lym loom status` names the producer in flight.**
  A shed run at its start, whose status file names no row yet, now takes the same "running" write the loop does after a failure.
  Before, the status named no row until the first row's verdict, so a long first row such as a writer showed as empty.

Closes #58
Closes #53
Closes #49

## Design decisions

- `discussed` is a status of its own, not a flag read on top of `draft`, so the layer, the renderer and the start guard each key on one value.
  A dependency on a discussed task weighs as one on a ready task, as for `draft`.
- The batten start guard now refuses a `discussed` task as it refuses a `draft` one, naming the status and the release command, since nothing should start a task whose Discussion is already done without a release.
- `Board.SetRunStatus` skips the write when the entry already carries the status, so repeated transitions at one row commit nothing to the board.
- The board write lives in the batten commit-status seam and runs before the no-op-transition skip, so a retry after a failed write is not swallowed by the marker.

## To check by hand

- The batten start guard ignores `discussed` on paths other than the first start (`internal/battencli/startguard.go`); the guard's `Unreleased` probe is only consulted for a run with no status file.
  Confirm a resume of a discussed task that has a leftover status file is acceptable.
- The equal-status skip in `Board.SetRunStatus` (`internal/boardengine/board.go`) is covered only through the batten seam's test, not by a direct `boardengine` test of the skip.
- Release a motif task live and watch `lym batten status`, `lym loom status` and the README together through Seed-Child, Run-Shed and Discussion-Write, then through approval into `discussed`.
  Nothing here ran a real motif task end to end.
