# Umbrella tasks: project threads that spawn and land from their own worktree

`orch-any-worktree` · Note · feature, fabric, batten, orch, board, reed, creel, shed, workspaces

## Why

The operator works long-term on a branch of their own in its own worktree, not on `main`, which in a large project is not theirs to drive; that worktree is where they want their orch.
Today every `lyx orch` verb refuses outside the hub's prime (`internal/orchcli/refusal.go`), every batten verb does too (`RefuseUnlessPrime` in `internal/battencli`), and orch keeps its files under the prime's `.lyx/orch/`.
Run parentage already follows the worktree a pair was created from (`aa67fd1d7`, #341), so a pair created from the operator's worktree would land there; orch and batten are what stand in the way.

## Want

- `lyx orch` and `lyx batten` run from any drivable worktree in the hub, which then acts as that orch's home: pairs branch from its branch and land into it, and orch's scratch, handoffs and reed strand live in that worktree.
- More than one orch per hub, one per home worktree, without stepping on each other's pairs, board claims or reed sessions.

## Open points

- What "prime only" protects today and whether any of it must stay prime-bound (board writes, deploy, `lyx board prune`, the hub workspace).
- What the relevant CONSTRAINTS.md invariants say about prime (batten's prime-anchored status and lock paths, Told-Geometry) and how they change.
- Who starts a task that becomes ready: presumably its umbrella's thread, found through the task's `parent`.

## Operator decisions (2026-10-18)

- A larger project is an umbrella task on the board: a long-lived fabric pair that is a starting point with its own orch thread.
  It is not a separate hub: it shares the hub's repo, weft, board, creel daemon and gate slots, and has its own `.lyx/`, thread and batten runs.
- The prime is the root umbrella.
- A task carries a `parent` field naming its umbrella's slug; empty means the prime.
  The parent's branch is derived from the slug and fabric's `branch_prefix` (so a work hub with `branch_prefix: hanf/` needs no thought about prefixes); no branch name is hard-coded or stored.
- A task's pair is created from its umbrella's branch and lands there, and the task's notices and escalations go to the umbrella's thread.
- The board is shared by the whole hub; an umbrella's thread sees and starts only its own tasks.
- An existing, spawned task can be turned into an umbrella: its pair and the commits on its branch stay, its loom run stops, a thread starts in the pair, and later tasks name it as `parent`.
  Since a run-held entry takes no board edits, this is a verb of its own that releases the run's lock.

## Operator views, not decisions (2026-10-18)

- At work the operator sets a branch of their own as their base and spawns large sub-projects from it, while smaller tasks still spawn from `main`.
- What the operator wants is a thread they can open in any worktree and hold a running conversation with, which can spawn everything from there.
  With two large projects at once, two such threads, one per project, which the operator talks to separately and which can talk to each other; they need not be called orchestrators.
- Millhouse supports this today: a thread anywhere creates a task and spawns it from its own worktree, the task records that thread and worktree as its parent, and it merges back there when done.
- The orch role should not be unique: any thread that starts another thread becomes its parent.
- Giving a thread the orch hosting (its own watcher and context cycling) is an explicit command; the hub orch is long-lived enough to need it, while a project thread is time-limited, so cycling matters less there but would be nice.
- Reed shows every session at once, which gets messy; the threads a project's thread starts should be grouped under it as one project.
- Nothing hub-wide is left for a thread: deploy and prune are in the Land row, and intake from GitHub issues runs on command.
- A thread that is not running is a real problem whatever the design, as it is for the orch today.
- Two hubs with two weft repos would work today, but would make exchanging information between them hard.

## Seen (2026-10-18)

- Batten's Worktree-Create calls fabric's `Add` from the prime (`internal/battencli/wire.go`), and `Add` forks from the HEAD of the worktree it runs from, so every batten run forks from the prime's checked-out branch.
- The parent is already per run after that: the pair's origin record carries `ParentBranch` and `ParentWorktree`, loom and darn take `params.parent` from it, parent locks are per parent branch, and the Land row skips the prime pull and deploy when the parent is not the prime's branch (`internal/battenshed/land.go`).
- The board has no field naming a task's parent today.
- Every file orch writes goes through one accessor, `internal/orchcli/paths.go`, under the prime's `<anchor>/.lyx/orch/`; `lyx orch` refuses outside the prime and its strand name carries no worktree (`agentname.RoleOrch`).

## Operator decision (2026-10-18): no process while an umbrella lives

An umbrella is not a shed run that waits for weeks, since nothing restarts a waiting process after a machine restart (today `lyx batten run <slug>` resumes from disk, but someone must run it).
It has work only at its start and its end:

1. Create: its pair, thread and workspace, in one short command.
2. Live: no process; it is a pair, a board task and a thread the operator talks to when they want.
3. Land: when the operator says the project is done, a short run does Merge-In, Publish, PR-Gate and Finalize into its own parent; that run may be a shed run, living only as long as the landing.

## Orch sketch, not a decision

- A thread starts its runs itself with `lyx batten run`, which finds the prime for the shared state; the run records the thread's strand as its owner, and notices go to that owner through creel rather than to a fixed `ly:orch`.
- Threads talk to each other through creel posts, as strands already can.
- This builds on `orch-split` (a Go run driver beside a conversational agent): once the run loop is Go, a thread holds only the conversation and its own runs' escalations.

## Orch recommendation, not a decision: shared groundwork with Weaver

Umbrellas and `weaver` need the same primitive: create a pair from a parent pair's branch, merge it back into the parent, tear it down.
Today fabric creates a pair only from the worktree it runs in, and batten only from the prime.
That primitive is its own piece of groundwork, built before either umbrellas or Weaver.
Weaver's batches are not umbrellas: they are not board tasks, run as cold agents for minutes rather than loom runs for days, merge locally with Haiku resolving conflicts, and have no conversational thread.
