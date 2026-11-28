# The arbiter takes budget escalations, and nothing lands a round with an open blocking finding unattended

`arbiter-budget-escalations` · Task · feature, arbiter, shed, loom, stencil · draft

Decision record: [Sketch](decision-arbiter-budget-escalations.md)

Depends on: [`driver-settles-stops`](design-driver-settles-stops.md)

Filed from the Sketch `self-healing-runs`.

Takes: D6's Go and arbiter-stencil steps: `Eligibility` and `Stop` in `internal/arbiterstop/arbiterstop.go`, built in `arbiterstop.Check` from `NewReaders`; the arbiter call in `internal/shedarbiter`; the stencil `contracts/stencils/loom/loom-template-arbiter.md`; the Bouncer's escalation and accept paths in `internal/shedadapters/bouncer.go` and `readCirclingFile` and `recordAutoContinue` in `internal/shedadapters/circling.go`; `circlingVerb` and `finalizeCirclingVerb` in `internal/loomcli/circling.go`; the fixer report's `deferred` disposition parsed by `internal/burlerengine/fixerreport.go`; `dobbyengine.LookupAutonomy` from autonomy-unattended-level.
Delivers: Cause `budget` admitted; `arbiterstop.Stop` carrying the run's level, named in the arbiter's prompt; the arbiter's `budget` rules in D6's order, with the one continue per segment read from the circling decision files and refused at the hard cap; one board note for a deferred-findings accept at Code-Review and after, filed by the driver through a seam the loomcli driver wiring backs with `internal/boardengine`, with no board grant for the arbiter; under `unattended` the Bouncer runs every perch as with `blocking_escalates`, and neither the arbiter at any cause nor a Finalize fix-review accept lands a round with an open BLOCKING finding or a red gate. The orch's side of D6, the hold and the role text, is built by orch-unattended-rules.
Exports: The `Autonomy` field on `arbiterstop.Stop`.

The part changes these packages:

- internal/arbiterstop (about 160 changed lines)
- internal/shedarbiter (about 90 changed lines)
- internal/shedadapters (about 230 changed lines)
- internal/sheddriver (about 160 changed lines)
- internal/loomcli (about 150 changed lines)
- contracts/stencils/loom (about 60 changed lines)
