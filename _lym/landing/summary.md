# Beam session: runs overview takes the slot, Beam-Up warnings surface

A beam's reed session could show pawl running with no orch and no beam overview.
Two defects in the overview slot and in Beam-Up's bring-up report caused it to go unseen.

## What changes

- A session admits one overview strand.
  When a pawl run's `lym loom start` added `loom-status` with `AnchorOverview` first, `ensureRunsStrand` was refused and only logged a warning, so the beam had no overview.
  Now `ensureRunsStrand` (`internal/orchcli/runs.go`) replaces a `loom-status` strand with the `runs` strand when no `runs` strand exists.
- In the other order, `lym loom start` in a beam whose session already holds a `runs` strand adds `loom-status` in a window of its own with `KeepViewer`, so it never takes the viewer (`statusStrandAddSpec` in `internal/loomcli/bootstrap.go`).
  Outside a beam the status strand keeps the overview window as before.
- `ensureRunsStrand` now returns a warning instead of only logging.
  `beamUp` appends it to `BeamUpReport.Warnings`, and each message names the cause and `lym beam start` as the way forward.
- The pawl run's Beam-Up row (`internal/loomcli/pawlwiring.go`) no longer drops the report.
  `runBeamUpStep` logs each warning and posts them as one notice to the beam's orch through the row's existing `Notify` seam.
  A warning never halts the row; an error from `BeamUp` still does.
- `loomengine.LoomStatusStrandName` and `orchcli.RunsStrandName` are exported so the two packages share one name for each role.
- `docs/overview.md` and the `loomcli` and `orchcli` package docs describe the shared overview rule.

Closes #11

## Design decisions

- The `runs` strand wins the overview slot in both orders, so the slot rule holds whichever bootstrap runs first.
- Warnings go to the orch as a notice rather than into the row's error path, since a missing overview is a degraded view, not a failed bring-up.
- Warning surfacing holds whichever cause is real, so it does not depend on the slot clash being the only cause.

## To check by hand

- The cause was derived from source; no live beam was observed.
  Attach to a beam's session mid-run and read `lym reed list`: a session attached before Beam-Up runs still has neither orch nor overview, and nothing tells the operator the orch comes later.
  This change does not alter that timing.
- Start a beam whose pawl run runs `lym loom start`, then run `lym reed attach` in the beam's directory after Beam-Up.
  Expect the `beam: <slug>` overview, the orch window, and the status strand in a window of its own.
- Check that the Beam-Up warning notice reaches the orch (`beamup-warning-surfacing-untested`, `internal/loomcli/pawlwiring.go`): `runBeamUpStep` is unit-tested with injected seams, but the post through `orchcli.NotifyOrchAt` was not exercised against a live orch.
