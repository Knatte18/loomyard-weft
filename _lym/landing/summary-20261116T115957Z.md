# lym beam pr open refuses a beam that conflicts with its parent

`lym beam pr open` could open a beam's pull request ready for review while the beam conflicted with its parent branch and a merge-in task still had to land.
GitHub showed it as dirty, and the operator could have merged a pull request whose work was still landing.

## What changes

Before `lym beam pr open` pushes for a pull request none records, or reopens a recorded one GitHub reports closed, it now refuses two beams:

- a beam with a merge task that is not done, which `lym beam merge-parent` filed;
  the refusal names `lym batten status <task>` and says to let the task land, since its landing changes what the pull request holds;
- a beam whose head conflicts with its parent branch;
  the refusal names `lym beam merge-parent <slug>`.

A beam that is only behind its parent still opens, since GitHub can merge it.
A recorded pull request that GitHub reports open or merged is handled as before.

The conflict test is `fabricengine.Fabric.ConflictsWith`, an in-memory `git merge-tree` of a ref into the code HEAD that changes no worktree, index or ref.
`hubLandDeps` exposes it as the new `ParentConflicts` seam.
That seam reads the same parent tips as `ParentUnmerged`: the local tip, and the upstream tip after a fetch when the parent has one and pushes run.
Both seams now share one `beamParentTips` reader.

## Design decisions

- The decision record offered three options: refuse, run the merge-in first, or open the pull request as a draft.
  The verb refuses.
  That keeps `pr open` a thin verb, and the merge-in already exists as a reviewed task through `lym beam merge-parent`.
- The check is a real conflict test, not the "parent holds commits the beam lacks" test `lym beam ready` uses.
  The operator's decision names conflicts, and a beam that is only behind can still be reviewed.
- An open merge task refuses even when no conflict shows.
  Its landing changes the beam, so a pull request opened before then would show work that is still landing, which is the reported case.

## Docs

The `beamcli` package doc and the orch's `contracts/orchrole/role-beams.md` describe the new refusals.

## Tests

- `TestPROpen` adds rows for an open merge task, a conflicting beam, and a closed recorded pull request on a conflicting beam.
  Each asserts the refusal's way forward, and that nothing was pushed, written, created, reopened or marked ready.
- `TestHubLandDeps_ReadyReads` drives `ParentConflicts` over real git: a parent commit on another file reads as no conflict, and both sides writing one file apart reads as a conflict.
