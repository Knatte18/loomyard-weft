# Let a filed arras run resume past Run-Shed and tear down

After an arras family filing, the filing turns the run's own board entry into that family's grouping project.
Every later `lym batten run` or `step` pass for that slug was then refused with "grouping project, only a beam project runs".
The run never reached Land or Worktree-Teardown, and the only way out was `lym batten cancel <slug> --remove`.
This was seen on `adopt-build-resplit`, where the inner loom run was done and batten stayed stuck at Run-Shed.
The bug is not specific to beams.

`startGuard.refuseProject` in `internal/battencli/startguard.go` now lets a grouping project through when its run is on record and carries a filing mark.
The guard then judges that resume with the task checks, as it would a task's.
A grouping project with no run on record, or whose run filed nothing, is still refused with the `lym beam create` way forward.

## Design decisions

- The guard reads the mark through a new `startGuard.FilingMarked` seam, wired in `hubStartGuard` to `shedrun.FilingMarked`.
  A nil seam reads every run as one that filed nothing, so existing guards keep their behavior.
- The mark only counts when the run is on record.
  A stray mark on a slug with no run does not unlock a grouping project.
- A failed read of the mark is refused, naming the probe and the rerun to try, like the guard's other probe failures.
- The change takes the first option from the board entry's idea, a guard that lets the filed run through.
  It does not make the filing finish the batten run.

Closes the board entry `filed-arras-stuck-at-run-shed`; it has no linked GitHub issue.

## To check by hand

- The hub wiring of `FilingMarked` in `hubStartGuard`.
  The unit table drives the seam directly, so the real `shedrun.FilingMarked` call is covered only by the new subtest in `internal/battencli/beamland_integration_test.go`, `GroupingProjectWhoseRunFiledItsFamilyResumesOnlyWithTheFilingMark`.
  That test is an integration test and was not run as part of this description.
- A real filed arras run past Run-Shed, resumed with `lym batten run`, reaching Land and Worktree-Teardown.
  This end-to-end path was not exercised.
