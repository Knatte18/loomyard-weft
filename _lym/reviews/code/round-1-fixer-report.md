# Fixer-report, code round 1

All six findings are fixed; no test failed. The gate is green: tier 0 over `hibernateengine`, `hibernatecli`, `fabriccli`, `beamcli` and `cleanupcli`, and tier 1 over `beamcli` and `hibernatecli`.

## Changes

- **F1** (fixed as proposed, second branch): `fabriccli` keeps its literal list, since the file's comment names the import chain that rules out a shared import.
  `hibernateengine` gains `TerminalPhases()` over one `terminalPhases` list that `Request.Terminal` now reads.
  `TestHibernateEndedPhases_AreTheEnginesTerminalPhases` in `internal/fabriccli/hibernatedaemon_test.go` fails when `hibernateEndedPhases` differs from that list in either direction.
  An `export_test.go` seam exposes the list; the seam is in `fabriccli`, since an `export_test.go` of `hibernateengine` cannot reach a `fabriccli` variable.
  The test writes the request through `hibernatecli.RequestPath`, so a rename of the file name breaks it as well.
- **F2** (fixed as proposed): `TestHibernateDaemonLive` covers no request file, a waiting request naming a live carrier, a pending request naming a dead carrier, a pending request with pid 0, and a done, failed and cancelled request still naming a live carrier.
- **F3** (fixed as proposed): `NewDobbyCarrier` creates the hibernate directory and returns the error.
  The assertion in `TestHubDobbyDeps_PrimeTidyShuttle` stats the directory before any tick, since a tick creates it as a side effect.
  Docs of `hibernatecli` updated.
- **F4** (fixed as proposed): the failed reason of a request taken up in the resuming phase says its wake reconcile may not have run and that `lym cleanup --apply` runs it.
  The takeover row of `TestTakeUp` asserts the notice names both.
  The `carry` comment and the `hibernateengine` package doc updated.
- **F5** (fixed with the proposal's second branch): the carrier's goroutine rewrites the request file as it stands once `running` is cleared, so the file watch runs the next tick.
  A `TestCarrier` row cancels by writing a fresh request while the first sequence waits, checks the file was touched after the sequence ended, and checks the next `Tick` carries the fresh request.
  The `hibernateengine` package doc updated.
- **F6** (fixed as proposed): the comment in `internal/cleanupcli/cleanup_integration_test.go` names the request's wake.

Commits: `c55b8061` (F1, F2), `3f2c17fb` (F3), `e2f4b7f0` (F4, F5), `50235ed6` (F6), and one follow-up commit for the self-check findings.

## Self-check

All four forks returned a report; their files sit beside this report as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness, MEDIUM: the F3 test passed without the fix, since `Tick` creates the directory. Fixed as proposed: the stat moved before any tick.
- fix-consistency, LOW: carrier doc omits the end-of-sequence rewrite. Fixed as proposed in the `hibernateengine` package doc.
- fix-consistency, LOW: `hibernatecli` doc and `NewDobbyCarrier` comment omit the directory creation. Fixed as proposed.
- fix-consistency, LOW: run-on wording of the resuming reason and the `carry` comment. Fixed: the reason is reworded and the comment names it.
- fix-consistency, LOW (named in the lens's text): `internal/beamcli/doc.go` could mention the end-of-sequence wake. Fixed differently: not changed, since its sentence "a written request wakes the tick that takes it up" stays true and the `hibernateengine` doc owns the end-of-sequence rule.
- fix-proposal, LOW: the F4 reason claims the reconcile did not run, which a kill after the reconcile makes false. Fixed as proposed: "may not have run".
- fix-proposal, LOW: the F1 test's hand-listed phases miss a new terminal phase. Fixed with the engine's `TerminalPhases()`.
- fix-affected-set, LOW: the request file name is not pinned. Fixed: the test builds the path through `hibernatecli.RequestPath`.

## Disputed

None.

## Deferred

None.
