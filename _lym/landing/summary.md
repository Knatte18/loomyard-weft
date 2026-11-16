# lym beam pr open refuses a beam that conflicts with its parent

`lym beam pr open` opened beam `adopt-build`'s pull request ready for review while it conflicted with `main` and while a merge-in member still had to land.
An operator could have squash-merged a pull request whose work was still landing.
`pr open` now refuses in both cases and names the way forward.

## What changes

Before it pushes the beam for a pull request none records, or reopens a recorded one GitHub reports closed, `pr open` calls the new `judgeOpenable` in `internal/beamcli/pr.go`.
It refuses in this order:

- A merge task of the beam that is not done: the refusal names `lym batten status <task>`, since the task's landing changes what the pull request holds.
- A beam whose head conflicts with its parent branch: the refusal names `lym beam merge-parent <slug>`.

A beam that is only behind its parent still opens, since GitHub merges it.
A recorded pull request that is already open, or merged, takes the paths it took before.

## Design decisions

- The change refuses instead of opening a draft pull request, the other option the board entry listed.
  The operator's decision was that a pull request must not be opened before its conflicts are fixed.
- The conflict test is `Fabric.ConflictsWith` in `internal/fabricengine/parenttips.go`.
  It merges the parent tip into the beam head in memory and leaves the worktree, index and refs untouched.
- The new `landDeps.ParentConflicts` reads the same parent tips as `ParentUnmerged`: the local tip and, when the parent has an upstream and pushes run, the upstream tip after a fetch.
  The tip reading moved into a shared helper, `beamParentTips`, in `internal/beamcli/deps.go`.
  The beam conflicts when either tip does.
- `lym beam pr open --help`, the `internal/beamcli/doc.go` package doc and `contracts/orchrole/role-beams.md` list the two refusals.

## To check by hand

- `lym beam pr open --help`: the help lists the merge-task and parent-conflict refusals, as the diff in `internal/beamcli/cli.go` now has it.
  A last review did not see the fixed text, so read that help text.
- Run `lym beam pr open` on a beam that conflicts with `main`, and on one with a merge task still open.
  This was not run against a real GitHub remote.
