# Restore the status pane when `lym loom resume` puts a driver to work

After a reboot, a run resumed with `lym loom resume` drove on with its `loom-status` strand dead.
The operator saw no overview pane and took the run for stuck.
`start` brings reed up and keeps, replaces or adds the status strand, but `resume` did neither.

Closes #253

## What changes

- `resume` now brings reed up and restores the status strand before it spawns a Go driver for a halted go-seeded run,
  and before it types the resume line into a parked driver.
  It does so under the bootstrap lock it already holds.
- A failed restore never fails the resume, since it costs the operator the view and not the run.
  It is logged at Warn, and the success envelope gains a `status_pane` field that names the error and `lym loom start`.
- A resume that is a no-op, and one refused before it reaches a wake or spawn, touch neither reed nor the strand.
  A wake that is refused after the restore, such as for a busy parked driver, leaves the pane restored.

## Design decisions

- The restore is a new `loomCLI.restoreStatusPane` seam, which `wire` sets to `reed.Up()` followed by `ensureStatusStrand`.
- It runs on both wake paths, not only the Go-driver spawn the issue reports:
  a parked live driver can also have a dead status strand, and a current live strand is kept.
- `ensureStatusStrand` now returns an error wrapping `errStatusStrandNotReplaced` on a failed replace,
  where it used to log and return nil.
  `start` logs that error and carries on as before, and `resume` reports it in `status_pane`.
- A refused wake carries no `status_pane`, so a restore failure on that path is only logged.
- `resume` never calls `FollowRun`.
  It moves a viewer only when the viewer sits on the status strand window the restore replaces:
  that window closes, and a successful replace moves the view to the replacement's window.

Help text, the `internal/loomcli` package doc and `docs/overview.md` describe the new behaviour.

## To check

- A failed status-strand replace on resume must not be silent (`ensureStatusStrand` in `internal/loomcli/sharedbootstrap.go`).
  After a reboot the restore takes the replace branch, so check that its error reaches `status_pane`,
  and that `start` (`internal/loomcli/start.go`) still treats it as non-fatal.
- The real restore seam must be tested, not only the fake in `TestResumeVerb`
  (`restoreStatusPane` in `internal/loomcli/wiring.go`, `internal/loomcli/smoke_drivingsurface_test.go`).
  Check that `TestSmokeStatusStrandAcrossDriverSeeds` asserts a live replacement strand after a real resume,
  and a `status_pane` warning when reed refuses the replace.
- The reported scenario has not been run by hand: reboot with a run in flight, run `lym loom resume`,
  and confirm the `loom-status` pane is back.

## Merge danger

**Door:** two-way
**Blast radius:** loom
