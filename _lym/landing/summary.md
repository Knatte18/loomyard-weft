# Restore the status pane when `lym loom resume` puts a driver to work

After a reboot, a run resumed with `lym loom resume` carried on with its `loom-status` strand dead.
The operator saw no overview pane and took the run for stuck.
`start` brings reed up and keeps, replaces or adds the status strand on every call.
`resume` did neither, so nothing recreated a strand the reboot had killed.

## What changes

- `resumeFromRunState` in `internal/loomcli/resume.go` restores the status pane before it puts a driver to work:
  before it spawns a Go driver for a halted go-seeded run, and before it types the resume line into a parked driver.
  It does this under the bootstrap lock it already holds, the same window `start` uses.
- The restore is a new `loomCLI.restoreStatusPane` seam.
  `wire` sets it to `reed.Up()` followed by the existing `ensureStatusStrand`, which keeps a current live strand, replaces a stale one and adds a missing one.
- A failed restore never fails the resume, because losing the pane costs the operator the view, not the run.
  The failure is logged at Warn, and the success envelope gains a `status_pane` field that names the error and `lym loom start`.
  `start` restores the pane over a live driver and leaves that driver working.
- Refusals and no-op outcomes, such as a done run or a running run with a live driver, still touch neither reed nor the strand.
  `TestLoomResumeReconcilesThenRefusesWithoutLiveDriver` still holds for the refusal path; only its wording changes.

## Design decisions

- The restore happens on both wake paths, not just the Go-driver spawn named in the report.
  A parked live driver keeps its tmux session, but its status strand can still be dead, and `reed.Up` plus `ensureStatusStrand` is idempotent over a live pane.
- `resume` still never calls `FollowRun`, so it still never switches a viewer's window.

## Tests

`TestResumeVerb` installs a counting `restoreStatusPane` fake.
Every row asserts that the restore runs exactly once when a driver is spawned or a resume line is attempted, and never otherwise.
The ordered rows now expect `pusher`, `pane`, `wake`.
A new row makes the restore fail and asserts the Go driver is still spawned, the envelope is a success, and `status_pane` names the error and `lym loom start`.
On the merge base the test fails, since the base has no restore seam.

Docs updated: the `resume` verb's help text and file header, the `# Resume` section of `internal/loomcli/doc.go`, and the loom passage of `docs/overview.md`.

## Merge danger

**Door:** two-way
**Blast radius:** loom
