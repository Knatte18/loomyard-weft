# Orch and batten run from the operator's own long-lived worktree, not only the prime

`orch-any-worktree` · Note · enhancement, fabric, batten, orch

## Why

The operator works long-term on a branch of their own in its own worktree, not on `main`, which in a large project is not theirs to drive; that worktree is where they want their orch.
Today every `lyx orch` verb refuses outside the hub's prime (`internal/orchcli/refusal.go`), every batten verb does too (`RefuseUnlessPrime` in `internal/battencli`), and orch keeps its files under the prime's `.lyx/orch/`.
Run parentage already follows the worktree a pair was created from (`aa67fd1d7`, #341), so a pair created from the operator's worktree would land there; orch and batten are what stand in the way.

## Want

- `lyx orch` and `lyx batten` run from any drivable worktree in the hub, which then acts as that orch's home: pairs branch from its branch and land into it, and orch's scratch, handoffs and reed strand live in that worktree.
- More than one orch per hub, one per home worktree, without stepping on each other's pairs, board claims or reed sessions.

## Open points

- What "prime only" protects today and whether any of it must stay prime-bound (board writes, deploy, `lyx board prune`, the hub workspace).
- How a home worktree's branch reaches `main` (the operator's own merge, or a landing verb), and whether the board is per hub or per home.
- What the relevant CONSTRAINTS.md invariants say about prime (batten's prime-anchored status and lock paths, Told-Geometry) and how they change.
