# Tell a re-spawned writer the interrupted spawn's commits and scratch

A darn writer spawn killed before it recorded an outcome, by a machine reboot for instance, leaves no Darn-Write history entry.
The next spawn's prompt then gave its prior work as `(none)` and said the branch held no work of its own.
In fact the task branch already carried the killed spawn's commits, and `.scratch` held its half-finished files.
A writer that trusted the prompt could redo or overwrite finished work.

## What changes

- `loomshed.DarnDeps` gains two seams. `BranchCommits` lists the task branch's commits beyond its merge base. `ScratchDir` is the directory the writer keeps scratch files under.
  The producer reads both before each spawn, following `ScratchDir` when it is a link, as the `.scratch` link is.
- `renderPriorWork` decides the prior work from those reads as well as the recorded history:
  - with no recorded outcome but commits beyond the base or files in the scratch directory, the note says an earlier spawn was interrupted before it recorded an outcome and its work may be present, then lists the commits and the files;
  - after a recorded outcome, the existing note stands and the same lists follow it;
  - it stays empty, rendered as `(none)`, only when there is no recorded outcome, no commit beyond the base and no scratch file.
  Each list shows its first 20 items and counts the rest.
- `loomcli.darnDeps` wires `BranchCommits` to `CommitLog` from the task's merge base to `HEAD`, and `ScratchDir` to the new `fabricengine.ScratchLinkPath`.
- The `DarnWrite` registry row refuses an unwired `Darn.BranchCommits` or a relative `Darn.ScratchDir`, as it does the other seams.

## Design decisions

- The rows before Darn-Write in the darn recipe commit only to the records side.
  So any code commit beyond the merge base at Darn-Write is the writer's own, and listing it needs no filter.
- The note states the interruption as a likelihood ("may be present"), since no record says what the killed spawn finished.
- Each list is capped so a scratch directory full of files cannot swamp the prompt.

## Tests

- `TestDarnWrite_Call` gains a regression row.
  A spawn with no recorded outcome, two commits beyond the base and two files under a linked scratch directory must be told it was interrupted, with both lists.
  On the previous code this spawn was told nothing.
  The test also gains rows for a failed commit read and for each unwired new seam.
- `TestRenderPriorWork` gains rows for a recorded outcome with commits after its findings, and for scratch files alone with a list past the cap.
- `TestDarnWriteEntry_ConstructionFailures` gains rows for the two new `Env.Darn` fields.

## Merge danger

**Door:** two-way
**Blast radius:** loomshed
