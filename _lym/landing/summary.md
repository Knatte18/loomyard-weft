# Raddle and motif runs start and finish without the orch

Three hand steps the orch had to take around a Sketch's raddle run are gone.

## What changes

**A promoted Sketch starts its raddle run with no cleanup.**
Promoting a Sketch gives the entry recipe raddle under the slug of the motif run that made it, and that run's done record under `_lym/shed/<slug>` made battencli refuse the start with `has already completed`.
The start of a raddle run now removes the slug's done motif run, as `lym batten cleanup --apply` does, and goes on.
The removal happens in `battencli`'s start guard after the start's owner check passes, so a task another machine owns loses nothing.
Dobby's read of a released task applies the same judgment, so it no longer holds the start for that run.
A done run of any other recipe, or a raddle entry whose done run was not a motif run, is still refused as before.
If the board or the child-recipe record cannot be read, the start is refused naming the command to re-run, and dobby keeps holding it.

**A filed raddle run goes on through Land and Worktree-Teardown.**
Raddle-File marks the entry done, and `fabricengine`'s origin reconcile then read the run as ended on origin and paused it at Run-Shed naming `lym cleanup --apply`.
The reconcile now exempts a raddle run standing at Raddle-File when origin shows its entry done, the way it already exempts a run whose landing is under way.
An entry origin removed before it ended done still reconciles as ended.
A pruned run the ledger had marked ended now reads as `done` in the ended set, not `removed`, which the exemption needs.

**A refused Sketch tells the run's parent.**
When the board refuses to make a finished motif run's entry a Sketch, as while another entry depends on it, `sketchFinish` in `loomcli` still leaves the entry a draft task, and now also mails the run's parent.
The notice names the board's refusal, which lists the dependent entries, and both ways forward: drop or move those dependencies and make the entry a Sketch, or give the task recipe raddle and release it.
A failed mail only logs a Warn.

`docs/overview.md`, the package docs and the `lym batten cleanup` help text state the new start behavior.
The records commit of `ClearMotifRun` now reads `batten: remove the done motif run of <slug>`, since two callers make it.

## Design decisions

- The raddle start removes the done motif run through the existing `ClearMotifRun`, so its refusals (lock held, run not done, child recipe not motif) stay in one place.
- The guard judges the start twice: once read-only before the owner check, which pulls the board, and once more before the removal, so a pulled board that no longer says raddle removes nothing.
- The Raddle-File exemption counts only a pair whose current producer is Raddle-File, never a history row, since a return sends the run back to Raddle-Write.

## To check

- The refused-Sketch notice's first way forward embeds a `lym board set-deps` command and a `lym board upsert` command for the Sketch.
  Run each as the notice prints it against a board with a dependent entry, in `sketchRefusedNotice` in `internal/loomcli/wiring.go`, to confirm both are runnable as written.
- On a live hub, promote a Sketch to raddle and release it, then confirm dobby starts the run with no cleanup and the run ends through Land and Worktree-Teardown.
  The new tests cover the pieces, not this whole path.
