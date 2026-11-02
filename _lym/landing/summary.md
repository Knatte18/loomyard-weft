# A beam's runs strand takes the overview; Beam-Up surfaces its warnings

Fixes [issue #11](https://github.com/Knatte18/loomyard/issues/11): a beam's reed session showed pawl running with no beam overview and no orch.

## Cause

Candidate 2 of the issue is confirmed from the code.
A beam's pawl run is a child run, and batten bootstraps every child run with `lym loom start --no-attach` in the child's anchor (`internal/battencli/wire.go`), so the pawl run's bootstrap runs in the beam's own reed session.
`lym loom start` adds the `loom-status` strand with `AnchorOverview` there, before the pawl recipe reaches Beam-Up.
Beam-Up's `orchcli.BeamUp` then calls `ensureRunsStrand`, whose add of the `runs` strand reed refuses, since a session admits one overview strand.
The refusal was only logged, and Beam-Up dropped `BeamUp`'s report, so nothing showed it.

The orch itself is launched after `ensureRunsStrand` whatever it returns, so the slot clash does not explain the missing orch.
That part matches candidate 1: a session attached before the pawl run reaches Beam-Up has no orch yet.
This change makes no timing change, but a Beam-Up failure that does stop the orch was already a halting Stuck, and its warnings now reach the orch.

## What changes

- **The runs strand holds the overview of a beam's session.**
  `ensureRunsStrand` (`internal/orchcli/runs.go`), finding no `runs` strand, replaces a `loom-status` strand through `ReplaceStrand`, which reed never refuses for the overview strand itself, instead of adding beside it.
  `lym loom start`'s `statusStrandAddSpec` (`internal/loomcli/bootstrap.go`) now takes the session's strands, and when a `runs` strand is among them, puts `loom-status` in a window of its own with `KeepViewer`, rather than having its add refused and the start fail.
  The rule holds in both orders: Beam-Up after a pawl bootstrap takes the overview, and a later `lym loom start` in the beam, such as a resume, adds its status strand beside it.
- **Runs strand failures are report warnings.**
  `ensureRunsStrand` returns a warning naming the failure, its cause and `lym beam start`, and `BeamUp` adds it to `BeamUpReport.Warnings`, so `lym beam start` and `lym adopt` show it as well.
- **Beam-Up surfaces the report.**
  The Beam-Up step in `pawlDeps` (`internal/loomcli/pawlwiring.go`) logs each warning of the report and posts them to the beam's orch as one `PAWL` notice; none halts the row, as before.
- **Shared names.**
  `loom-status` is now `loomengine.LoomStatusStrandName`, which `loomcli` and `orchcli` both read, and the runs role is exported as `orchcli.RunsStrandName`.

## Design decisions

- Of the issue's two ways to share the overview, `ensureRunsStrand` replaces the `loom-status` holder.
  Sending `loom-status` elsewhere only at its add cannot work alone: the pawl bootstrap runs before Beam-Up, when the pair is not yet marked a beam and no `runs` strand exists.
  The add-side rule is still needed so a later `lym loom start` in a beam does not fail on the refused overview add.
- `ensureRunsStrand` cannot tell a `loom-status` strand's anchor from reed's strand status, so it replaces one in its own window too; the next `lym loom start` adds it back beside the runs strand.
- A dobby that did not start stays a warning in Beam-Up, not a halt, matching `BeamUp`'s own rule for every other caller.

## Tests

- `TestStart_EnsuresTheRunsStrandBeforeTheOrch` gains a row where a live `loom-status` strand is replaced by the runs strand, and its failing-add row now pins the warning on the report.
- `TestStatusStrandAddSpec` pins the overview display with no strands or unrelated ones, and the own-window display beside a `runs` strand.
- `TestBeamUpWarningLine` pins the notice text, and that no warnings post nothing.
