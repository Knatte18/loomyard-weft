# A filed arras run resumes under its family's project and tears down

A family filing turns the run's own board entry into the family's grouping project.
Batten's start guard refused any run or step of a non-beam project with "grouping project, only a beam project runs".
So every pass after the filing was refused at Run-Shed, and the run never reached Land or Worktree-Teardown.
The workaround was `lym batten cancel <slug> --remove`.

`refuseProject` in `internal/battencli/startguard.go` now reads the run's filing mark through a new `FilingMarked` guard seam, wired to `shedrun.FilingMarked` in the prime.
A grouping project whose run is on record and whose filing mark stands is handed to the task checks as a resume, which pass it past Worktree-Create.
A failed read of the mark refuses naming the probe and the re-run.

## Design decisions

- The pass needs both a status file and the mark.
  A fresh run of a grouping project is still refused even if a stray mark is on disk, so the mark lets through only the run that wrote it.
- The guard lets the run through rather than having the filing finish it.
  The run's remaining rows, Land and Worktree-Teardown, then run as they do for any task, and `ClearMotifRun` removes the done arras run as before.

## Tests

`TestStartGuard_Refuse` gains rows for:

- a filed run resuming under its family's project, dirty source and left pair parts included;
- an unfiled run of a grouping project, still refused;
- a filing mark with no run on record, still refused;
- a failed mark read, which refuses naming the probe.
