# fabric: Publish merge-in fails with 'gitrepo: worktree status: object not found' and leaves a half-staged merge

`merge-in-object-not-found` · Note · bug, fabric, loom

Imported from [issue #468](https://github.com/Knatte18/loomyard/issues/468).

## Seen

Run `binary-change-and-rules`, Publish, 2026-10-08T18:41Z (trace `trace-20261008T184138Z-82c70335c18c4f83-634197.log` in the task worktree).
After `fabric: mutation kind=merge_staged` and a conflict agent finishing `done`, Publish failed with:

`landingshed: Publish: merge-in against parent branch "main": mergeresolve: list untracked files (attempt 1): fabricengine: list untracked files: gitrepo: worktree status: object not found`

The worktree was left mid-merge with `internal/battencli/wire.go` unmerged (UU) and every merged path staged.
The driver's repair re-step then stopped `blocked` on "the task worktree has uncommitted changes before the merge-in", naming the staged merge itself.
`git fsck --connectivity-only` on the pair was clean, so the object exists for git; the failure is in lyx's own worktree status read during a merge.
The machine had been hard-reset about 30 minutes earlier; that may or may not matter.

## Workaround

The orch ran `lyx fabric merge --abort` in the task worktree and `lyx loom resume`, so Publish redoes the merge-in from a clean tree.

## Expected

The untracked-file listing works mid-merge, or a failed merge-in aborts its own staged merge before refusing, so the run never halts on its own leftover merge, and the dirty-tree refusal names `lyx fabric merge --abort` when a merge is in progress.
