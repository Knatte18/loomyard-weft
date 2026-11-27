# darn: tell a re-spawned writer what an interrupted spawn left behind

A `Darn-Write` run interrupted before it recorded an outcome, for instance by a reboot, was told its prior work was `(none)`
when it was stepped again, though the task branch already carried the earlier spawn's commits.
A writer trusting that could redo or overwrite finished work.

The prior-work note is now built from the branch and the scratch directory as well as from the latest recorded outcome:

- With a recorded outcome, the note names it and any kept gate findings, as before.
  When there are commits or scratch entries to list, it adds that a later spawn may have been interrupted,
  so the lists can go beyond the spawn that ended that way.
- With no recorded outcome but commits beyond the merge base, it says an earlier spawn was interrupted
  and its work may be present.
- With only scratch entries, it lists them and says an interrupted spawn or another agent may have left them.
- With none of the three, the prior work is still `(none)`.

Every non-empty note lists the commits and scratch entries there are,
and ends with the existing line on recorded decisions winning over findings.
Step 2 of the darn writer's stencil is reworded to match:
it no longer says the commits on the branch are the writer's own,
and tells it to read the listed work when a spawn was or may have been interrupted.

Closes #278

## Design decisions

- **Scratch entries alone claim no interrupted spawn.**
  Other agents of the task write to `.scratch` too, so files there are not evidence of an earlier writer.
  They are still listed: for a spawn killed before its first commit, the note has nothing else of it to list.
- **The scratch listing is shallow.**
  Only direct children are listed, a directory with a trailing slash, so one deep tree cannot crowd out other entries.
  The scratch directory itself may be a link, which is followed, and an absent or dangling one holds nothing.
  A child that is a link is listed without a slash and not followed.
- **Each list is capped** at `priorWorkListLimit` items, with the rest counted.
- **A failed read halts the row before the spawn.**
  This covers both the branch's commit log and the scratch directory itself.

`DarnDeps` gains `BranchCommits` and `ScratchDir`, which the producer and the `DarnWrite` recipe entry both require.
`darnDeps` in `loomcli` wires them from the task's merge base and the new `fabricengine.ScratchLinkPath`.

## To check by hand

- No test interrupts a real run and steps it again.
- The wiring of `BranchCommits` and `ScratchDir` in `darnDeps` (`internal/loomcli/darnwiring.go`):
  check that `TestDarnDeps_BranchWorkSeams` pins the base, the range and the path well enough.
- The scratch-only note from `renderPriorWork` (`internal/loomshed/darnwrite.go`):
  it claims no interruption but is not `(none)`, so a first spawn can receive it.
  Check that its wording cannot lead that writer to treat another agent's files as its own work.
- Step 2 of `contracts/stencils/darn/darn-template-write.md`:
  it says to continue from an earlier spawn's work whenever the prior work is not `(none)`.
  Check that this holds for the scratch-only note, which claims no earlier spawn's work.

## Merge danger

**Door:** two-way
**Blast radius:** darn
