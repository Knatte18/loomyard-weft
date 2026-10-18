# [WS 004] Board projects and pairs created from a parent pair

`pair-from-parent` · Note · feature, fabric, batten, board, workspaces

## Seen (2026-10-18)

- Fabric's `Add` forks a pair from the HEAD of the worktree it runs in (`internal/fabricengine/add.go`), and batten's Worktree-Create calls it from the prime (`internal/battencli/wire.go`), so every batten run forks from the prime's checked-out branch.
- Every batten verb refuses outside the prime (`RefuseUnlessPrime` in `internal/battencli`).
- After creation the parent is already per run: the origin record carries `ParentBranch` and `ParentWorktree`, loom and darn take `params.parent` from it, parent locks are per parent branch, and the Land row skips the prime pull and deploy when the parent is not the prime's branch (`internal/battenshed/land.go`).
- The board has no field grouping tasks or naming where a task's pair comes from.

## Operator decisions (2026-10-18, recorded in `orch-any-worktree`)

- A task carries one `project` field naming a project entry on the board.
- A project always groups its tasks, and the project entry says whether it has a worktree of its own; with one it is an umbrella, without one its tasks go to the prime.
- The project entry may hold the ordered list of its tasks, so the README shows it as a plan; the `workspaces` work is the first project, with no worktree of its own.
- An umbrella's branch is derived from its slug and fabric's `branch_prefix`; no branch name is hard-coded or stored.
- A task of an umbrella has its pair created from the umbrella's branch and lands there.

## Idea

The groundwork umbrellas need, built first:

- Board: project entries (with their own-worktree flag and ordered task list), the `project` field on tasks, and README grouping by project; useful at once for grouping-only projects such as `workspaces`.
- Fabric creates a pair from a named parent pair's branch rather than from the worktree it runs in.
- Batten's Worktree-Create creates the pair from the task's project when that project has a worktree, and batten verbs run from any worktree of the hub, keeping their shared state under the prime.
- Merge-back into the parent and teardown work unchanged, as they already follow the origin record.

`weaver`'s per-batch pairs need the same create-from-parent-pair, merge-back and teardown primitive.

## Folded in: board slug check and blocked-run cleanup (issue #724, operator decision 2026-10-18)

- Seen 2026-10-18: task `orch-rules-in-weft` was promoted and started, and batten blocked at Worktree-Create with an invalid slug, since a slug ending in `-weft` is reserved for weft worktrees.
  The orch cleared the run claim with `lyx board set-status`, renamed the entry to `orch-rules-files` with `lyx board merge` and started a new run; the old blocked batten record `_lyx/shed/orch-rules-in-weft` stays, since `lyx batten cleanup` skips blocked runs.
- Expected: `lyx board upsert` and `promote` refuse a task slug fabric cannot use, naming the rule, from one slug rule fabric owns; and a blocked run whose board entry is gone can be cleaned up.
