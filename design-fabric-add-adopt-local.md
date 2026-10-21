# fabric add refuses an existing local branch instead of adopting it

`fabric-add-adopt-local` · Note · bug, fabric, workspaces

## Seen (orch, 2026-10-21)

`lyx fabric add <slug>` adopts a pair whose warp branch exists only on origin, and a weft branch that exists locally or on origin.
It refuses with `ErrBranchExists` when the warp branch exists locally without the local-only mark (`internal/fabricengine/add.go`, `add`).
So there is no lyx way to make a pair over a branch that already lives in this clone, such as `main` or an umbrella branch, and the operator falls back to a plain `git worktree add` that lyx does not see.

Case: on the Models hub, as a stopgap until umbrella tasks exist, the operator wants the prime on an umbrella branch and `main` in its own worktree `main`.
`lyx fabric add main` is refused, because `main` exists locally.
It would also fail on the weft side, since `main-weft` is checked out in prime-weft, and git refuses one branch in two worktrees.

## Idea

`lyx fabric add` adopts an existing local warp branch the same way it adopts one from origin, instead of refusing.
The weft side adopts or forks `<branch>-weft` as it does today.
A branch already checked out in another worktree is refused with a message naming that worktree and `lyx fabric checkout` to move it off.
The adopted pair's recorded parent needs a decision: a pair over `main` or an umbrella branch has no parent in the task sense, so it may need to be recorded as a root pair that batten never lands.
