# Scratch link wiring, reconcile, remove and cleanup

Batch 01 (`01-scratch-link-wiring`) is done at `7dbb0446`.
It added the scratch step in `internal/fabricengine/scratchlink.go` with its integration tests in `scratchlink_integration_test.go`.
Its digest lists both files as deviations; both are new files the card creates.

Batch 02 (`02-reconcile-converges-scratch`) is done at `f9bee5c0`.
It covers cards 2, 3 and 4: reconcile converging the scratch link, `lyx fabric remove` unlinking it, and cleanup working through it, with the Windows check joining the `fabric-windows` board entry.
Its digest lists `scratchlink_integration_test.go` as a deviation, a file batch 01 created that the later cards extended.

Neither batch reported departures or warnings, and the run was paused hub-wide between the batch 02 fork finishing and its `record-batch`, then resumed.
