# Orch and batten run from the operator's own long-lived worktree, not only the prime

`orch-any-worktree` · Note · feature, fabric, batten, orch

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

## Operator view, not a decision (2026-10-18)

At work the operator sets a branch of their own, such as `henrik`, as their base and spawns large sub-projects from it, while smaller tasks still spawn from `main`.
So one orch needs to start runs from more than one parent branch, not only one orch per home worktree.

## Seen (2026-10-18)

- Batten's Worktree-Create calls fabric's `Add` from the prime (`internal/battencli/wire.go`), and `Add` forks from the HEAD of the worktree it runs from, so every batten run forks from the prime's checked-out branch.
- The parent is already per run after that: the pair's origin record carries `ParentBranch`, loom and darn take `params.parent` from it, parent locks are per parent branch, and the Land row skips the prime pull and deploy when the parent is not the prime's branch (`internal/battenshed/land.go`).

## Operator view, not a decision (2026-10-18, later)

- What the operator wants is a thread they can open in any worktree and hold a running conversation with, which can spawn everything from there, whatever the worktree.
- With two large projects at once, two such threads, one per project, which the operator talks to separately and which can talk to each other.
- They need not be called orchestrators.
- Two hubs with two weft repos would work today, but would make exchanging information between them hard.

## Orch sketch, not a decision

- Split the hub machinery from the conversation: batten runs, the board, creel, landing, deploy, prune and intake stay hub-level and prime-anchored, while the thread is an ordinary strand in its worktree that asks the hub to start runs.
- A run started from a thread takes the thread's branch as its parent and records the thread's strand as its owner; batten's notices and escalations go to that owner through creel rather than to a fixed `ly:orch`.
- Threads talk to each other through creel posts, as strands already can.
- This builds on `orch-split` (a Go run driver beside a conversational agent): once the run loop is Go, a thread holds only the conversation and its own runs' escalations.

## Orch recommendation, not a decision (earlier)

A per-task parent branch, as a board field (and a `lyx batten run --parent` override), from which Worktree-Create forks the pair (`origin/<parent>`) instead of the prime's HEAD.
One orch, one board and one prime then serve runs on `main` and on `henrik` alike, and the multi-orch part of Want becomes optional.
