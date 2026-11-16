# A filed arras run is refused at Run-Shed and never tears down

`filed-arras-stuck-at-run-shed` · Task · bug, batten

## Seen (adopt-build beam orch, 2026-11-16)

After an arras family filing, the batten run whose pass ended at Run-Shed is refused on every later pass with "grouping project, only a beam project runs", so it never reaches Land or Worktree-Teardown.
Seen on `adopt-build-resplit`: the inner loom run was done, batten stuck at Run-Shed.
The workaround was `lym batten cancel <slug> --remove` from the beam; the task branch had no commits.

## Cause

`internal/battencli/startguard.go` `refuseProject` refuses any non-beam project entry and never checks the run's filing mark (`shedrun.FilingMarked`),
though `cleanup.go` `ClearMotifRun` expects a filed arras run to finish.
Not beam-specific.

## Idea

`refuseProject` lets a slug with a standing filing mark through so the done run tears down, or the filing itself finishes the batten run.
