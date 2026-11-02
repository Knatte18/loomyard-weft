# A beam's reed session shows pawl running with no orch and no beam overview

`beam-attach-no-orch` · Task · bug, reed, orch, pawl, workspaces · running · Record-Write

Imported from [issue #11](https://github.com/Knatte18/loomyard/issues/11).

Seen in the Models hub, reported by the operator.

A beam is started for a project, and the operator runs reed attach in the beam's directory.
The session shows pawl running and no orch strand.

Expected: the orch and the beam overview, as lym beam start brings up the beam's dobby and its orch.

## Investigation

Read from the source only: no live beam existed to observe, so the cause is unconfirmed.

`lym beam create` starts only the pawl run (`internal/beamcli/create.go`).
The orch and the beam overview come from the pawl run's Beam-Up row, which calls `orchcli.BeamUp` (`internal/loomcli/pawlwiring.go`) and so runs the dobby, `ensureRunsStrand` and `startOrch` in order (`internal/orchcli/start.go`).
The overview is the `runs` strand with `AnchorOverview` and the window title `beam: <slug>` (`internal/orchcli/runs.go`); the orch is an own-window strand.

Candidate causes:

1. Timing, likely for the missing orch: Beam-Up runs after Preflight and Record-Write, so a session attached before it has neither orch nor overview, and nothing tells the operator the orch comes later.
2. The overview slot is taken: a session admits one overview strand (`internal/reedengine/replace.go`), and `lym loom start` adds `loom-status` with `AnchorOverview` (`internal/loomcli/bootstrap.go`).
   When `loom-status` comes first, `ensureRunsStrand` is refused and only logs a warning that the overview will be missing (`internal/orchcli/runs.go`).
   Not verified: whether the pawl run runs `lym loom start` in the beam's own session.

Beam-Up drops the report `BeamUp` returns (`internal/loomcli/pawlwiring.go`), so a refused dobby start or a refused overview add never stops the row and never shows.

Settling observation: attach to a live beam session mid-run and read `lym reed list` and the run's producer from `lym batten status`, to tell timing from the slot clash.

## Fix

1. Give `loom-status` and the `runs` strand one shared rule for the overview slot in a beam's session: the `runs` strand takes it and `loom-status` goes elsewhere, or `ensureRunsStrand` replaces a `loom-status` holder.
2. Beam-Up surfaces the warnings in `BeamUp`'s report, a refused overview add included, instead of dropping them.

## Run instructions

The cause is unconfirmed, so the writer first confirms from the code whether the pawl run runs `lym loom start` in the beam's own session, and so whether candidate 2 can happen.
Fix 2 holds whichever candidate is confirmed, since it surfaces what Beam-Up drops today; fix 1 follows only if candidate 2 is confirmed.
When it cannot confirm candidate 2, it does fix 2 alone and reports that fix 1 is unproven.
