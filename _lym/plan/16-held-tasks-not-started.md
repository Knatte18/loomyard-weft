# Card 16 — held-tasks-not-started

**Edit:**
- `internal/dobbyengine#TaskView`
- `internal/dobbyengine#readyToStart`
- `internal/dobbyengine#daemon.startReady`
- `internal/dobbyengine#WaitingTask`
- `internal/dobbyengine#BuildView`
- `internal/dobbyengine#TestReadyToStart`
- `internal/dobbyengine#TestBuildView`
- `internal/battencli#DobbyTasks`
- `internal/battencli#battenCLI.refuseStartOwnedElsewhere`
- `internal/battencli#TestRefuseStartOwnedElsewhere`

**Uses:**
- `plan:internal/boardengine#Owners.HoldOf`

**Intent:** Claim no queued task under a hold, on any machine (D5: "under an adopt hold no queued task is claimed").

A queued task has no run yet, so the act rule's `Held` never reaches it; the start paths read the hold themselves.
`TaskView` gains `HeldBy`, the machine of the hold that holds the task, empty when none does; `DobbyTasks` fills it through `Owners.HoldOf` over the local Bolt it already reads, so an `adopt-fix` hold's fix task is not held.
`readyToStart` reads a held task as not ready, and `startReady` starts none, whatever slot is free.
`WaitingTask` gains `held_by`, which `BuildView` fills, so `lym beam status` shows why the task waits.

`lym batten run <slug>` of a released task refuses in `refuseStartOwnedElsewhere`, after its Bolt pull, when the hold holds the slug: `the hub is held by an adopt on machine <m>; way forward: wait for that adopt's hold to clear, then run lym batten run <slug> again; if machine <m> has stopped, ask the operator`.
Nothing names `--take-hold`.

Tests, tier0:
`TestReadyToStart` gains a held row and a fix-task row.
`TestBuildView` gains a waiting task carrying `held_by`.
`TestRefuseStartOwnedElsewhere` gains a held row asserting the refusal and its way forward, and a fix-task row that starts.

**ImpactSummary:** One TaskView field read by the start step and the waiting view, and one batten start refusal; the dobby of every worktree and batten's start honor it.
