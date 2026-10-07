# run-recovery: wait visibility and run recovery

All three batches reported `done`, covering cards 1 to 17.

## Batch 01 (cards 1–6), head `2b6678422`

- Reed gained its one op that sets or clears a strand's pane wait mark, with its PATTERN entry.
- The default status line renders marked panes with elapsed time through a `waits` token.
- `--unless-name` skips the add over a dormant strand too.
- Reed's sessionless directory reports a retiring strand.
- Reed resume drops a non-live strand whose done-when paths all exist.
- Shuttle sets an ungated run's output files as its strand's done-when list.
- Deviations: `internal/reedengine/statusline_test.go`, `internal/reedengine/state_test.go` and `internal/shuttleengine/startup_test.go` changed, because `Strand` became non-comparable and the default template gained the `waits` placeholder.

## Batch 07 (cards 7–11), head `f8be0b201`

- The verify shell runs in its own process group and is killed whole on cancel or expiry.
- `verifytree.Verify` takes a timeout, and an expiry is a failed verify rather than an error.
- Each verify gate reports a timeout as a failed gate naming it.
- Loom status renders a running verify as producer, elapsed time and command.
- Shuttle marks gate evaluation and the background-shell wait on disk and on screen.
- Deviations: `internal/shuttleengine/run.go` and `internal/shuttleengine/fixture_test.go`.

## Batch 12 (cards 12–17), head `293d592b2`

- Loom status reports shuttle's waits.
- The landing merge-in verify marks the driver strand through a told callback.
- `lyx loom resume` wakes a parked driver and refuses everything else with a way forward.
- The orch role stencil names `lyx loom resume` for every halted-child resume.
- Batten's spawn confirmation is per process, and batten revives a halted or awaiting child's dead driver once per episode without resuming the child.
- Deviations: `internal/loomcli/landingdeps_integration_test.go`, `internal/loomcli/landingdeps_markdone_test.go`, `internal/loomcli/landingdeps_verify_test.go`, `internal/loomcli/wiring_publish_integration_test.go`, `internal/loomcli/wiring_seams_test.go` and `internal/testkit/envkit/envkit.go`.
