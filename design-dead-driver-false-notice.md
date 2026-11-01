# Dobby sends a dead-driver notice for a run that has just started

`dead-driver-false-notice` · Note · bug, dobby

## Seen (hub orch, 2026-11-01)

Dobby sent `[dobby notice] pr-continue-wakes-row: running at Preflight; driver strand is dead; since 2026-11-01T06:18:50Z; history 0` three seconds after the batten run reached Run-Shed (06:18:50), while the inner run was still starting.
A few seconds later the inner run was at Darn-Write with its `ly:pr-continue-wakes-row:darn` strand live and no fault; the run is go-driven and has no driver strand.
The orch had to investigate a stop that never was.

## Expected

Dobby sends a dead-driver notice only for a run whose driver should be alive and is not: never for a go-driven run that has no driver strand, and never in the start-up window before the inner run's first step.
