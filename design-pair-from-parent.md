# [WS 004] Pairs created from a parent pair: board parent field, batten from any worktree

`pair-from-parent` · Note · feature, fabric, batten, board, workspaces

## Seen (2026-10-18)

- Fabric's `Add` forks a pair from the HEAD of the worktree it runs in (`internal/fabricengine/add.go`), and batten's Worktree-Create calls it from the prime (`internal/battencli/wire.go`), so every batten run forks from the prime's checked-out branch.
- Every batten verb refuses outside the prime (`RefuseUnlessPrime` in `internal/battencli`).
- After creation the parent is already per run: the origin record carries `ParentBranch` and `ParentWorktree`, loom and darn take `params.parent` from it, parent locks are per parent branch, and the Land row skips the prime pull and deploy when the parent is not the prime's branch (`internal/battenshed/land.go`).
- The board has no field naming a task's parent.

## Operator decisions (2026-10-18, recorded in `orch-any-worktree`)

- A task carries a `parent` field naming its umbrella's slug; empty means the prime.
- The parent's branch is derived from the slug and fabric's `branch_prefix`; no branch name is hard-coded or stored.
- A task's pair is created from its umbrella's branch and lands there.

## Idea

The groundwork umbrellas need, built first:

- A board `parent` field on tasks.
- Fabric creates a pair from a named parent pair's branch rather than from the worktree it runs in.
- Batten's Worktree-Create creates the pair from the task's `parent`, and batten verbs run from any worktree of the hub, keeping their shared state under the prime.
- Merge-back into the parent and teardown work unchanged, as they already follow the origin record.

`weaver`'s per-batch pairs need the same create-from-parent-pair, merge-back and teardown primitive.
