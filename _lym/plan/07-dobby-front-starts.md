# Card 7 — Dobby starts a front task first and settles its action

**Create:**
- `plan:internal/dobbyengine#TestStartReady_FrontTask` -> `func TestStartReady_FrontTask(t *testing.T)`

**Edit:**
- `internal/dobbyengine#TaskView`
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#daemon.startReady`
- `internal/dobbyengine#daemon.tick`
- `internal/dobbyengine#WaitingTask`
- `internal/dobbyengine#WaitingTask.Ready`
- `internal/dobbyengine#BuildView`
- `internal/dobbyengine#passFake`
- `internal/dobbyengine#newPassHarness`
- `internal/dobbyengine#TestBuildView`
- `internal/dobbyengine/doc.go`
- `internal/battencli#DobbyTasks`
- `internal/battencli#DobbyLines`
- `internal/battencli#TestDobbyLines`
- `internal/beamcli#hubDobbyDeps`

**Extends:**
- `PATTERN.md`
- `internal/beamcli/doc.go`

**Uses:**
- `internal/dobbyengine#readyTask`
- `internal/dobbyengine#passHarness.tick`
- `plan:internal/fronttask#Settle`
- `plan:internal/fronttask#HubSettleDeps`
- `plan:internal/fronttask#IsOpen`

**Intent:**
D8's start hold and land action in dobby.

`TaskView` gains `Front bool` and `Project string`, read for every entry; `battencli.DobbyTasks` fills both from the board entry.
An open front task of this dobby's worktree is a task entry whose `Front` is true, whose status is empty or a run status, and whose `Project` is `d.geom.Slug` (empty for the prime's dobby), the rule `fronttask.IsOpen` states; dobbyengine judges it from the view with `boardengine.IsEmptyStatus` and `boardengine.IsRunStatus`, since it imports no `fronttask`.

`daemon.startReady`:
- while an open front task of the worktree stands, starts no other task, whatever max_runs leaves;
- starts that front task, when it is released, ready by `readyToStart` and not in flight, even when the runs in flight fill `max_runs`, held runs included (D4), before the loop that stops at `max_runs`; once started it takes its slot like any run, through `runsInFlight`.
  `max_runs: 0` still starts nothing, and the task-limit hold, a covering pause and the start backoff hold it as any task.
Bound: the hold reaches only new starts in this dobby's worktree, never a nested beam's (its own dobby reads its own project); a worktree runs at most one front task past `max_runs` at a time, since `fronttask` files none while one stands open; a draft front task is closed and lifts the hold.

`Deps` gains `SettleFronts func() error`, nil settling nothing.
`daemon.tick` calls it after `drivePasses` and the notices and before `startReady`, in the no-anchor-change branch, so a front entry with no record is recorded before its start.
An error stops nothing: the first of an episode is logged and posted once to the nearest live orch through `Deps.Notice` as `[dobby notice] front task: <error>`, the error naming the slug, the action and its hand step, such as `lym adopt <subject>`; the tick that settles cleanly ends the episode, and a later failure posts again.
`beamcli.hubDobbyDeps` fills it with `fronttask.Settle` over `fronttask.HubSettleDeps(prime, geom.Slug)`.

`WaitingTask` gains `FrontFirst string` (`json:"front_first,omitempty"`), the slug of the open front task that holds a released non-front task; `WaitingTask.Ready` reads false while it is set; `BuildView` sets it through the same rule `startReady` applies, and does not count a waiting front task against `max_runs`.
`battencli.DobbyLines` renders it as `front task <slug> runs first`.
`beamcli/doc.go` gains the line that the dobby's settle seam is `fronttask.Settle` over the worktree's front tasks.

Tests:
- `TestStartReady_FrontTask` over `newPassHarness`, whose `passFake` gains the front settle calls and whose harness fills `SettleFronts`: with `max_runs` running runs and a released open front task of the worktree, the front task starts and the other ready tasks do not; with an open front task running, no released task starts; a draft front task holds nothing; a front task of another project, its `Parent` another worktree than the harness's `prime`, holds nothing and does not start; the settle seam is called before the start in the same tick; under `max_runs: 0` the front task does not start; a pause covering the front task's slug holds it and the other tasks stay held behind it; a front task whose start backoff has not run out waits; a settle seam that fails posts one notice naming its hand step, posts none on the next failing tick, and the tick still starts the front task.
- `TestBuildView` gains a row: a released task under an open front task reads `FrontFirst` and not ready.
- `TestDobbyLines` gains the row of a task with `FrontFirst`.

`dobbyengine/doc.go`'s start section states the front-task hold, the start past `max_runs` and the settle step.

`PATTERN.md` gains, in `## Shed and loom` after `PATTERN-dobby-orch-pair`:

```
- `PATTERN-front-task` — Only fronttask files a front task and writes `front` and `on_land`; while one is open (empty or run status) dobby starts no other task in its worktree, and runs its `on_land` action from fronttask's closed table, on the subject's owner only; its project, kind and slug are fixed.
```

**ImpactSummary:** startReady gains the front hold and start, tick one seam call, the view one reason; one reader each in battencli and beamcli; tasks with no front entry start as before.

**Commit:** 7: dobby starts a front task ahead of the others and runs its land action
