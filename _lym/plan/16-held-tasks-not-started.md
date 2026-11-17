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

**Uses:**
- `internal/boardengine#Owners.HoldOf`

**Intent:** Claim no queued task under a hold, on any machine (D5: "under an adopt hold no queued task is claimed").

A queued task has no run yet, so the act rule's `Held` never reaches it; the start paths read the hold themselves.
`TaskView` gains `HeldBy`, the machine of the hold that holds the task, empty when none does; `DobbyTasks` fills it through `Owners.HoldOf` over the local Bolt it already reads, so an `adopt-fix` hold's fix task is not held.
`readyToStart` reads a held task as not ready, and `startReady` starts none, whatever slot is free.
`WaitingTask` gains `held_by`, which `BuildView` fills, so `lym beam status` shows why the task waits.

`lym batten run <slug>` under a hold is refused by `refuseByAct`, which `startFirstPass` runs first, with card 13's held refusal; `TestAct` and `TestReadRun` cover it.
The fix task's start passes because `ReadRun` reads it not held.

Tests, tier0:
`TestReadyToStart` gains a held row and a fix-task row.
`TestBuildView` gains a waiting task carrying `held_by`.

**ImpactSummary:** One TaskView field read by the start step and the waiting view; the dobby of every worktree honors it, and batten's start is refused by the act rule's hold.
