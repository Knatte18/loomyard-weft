# The driver and the arbiter settle known stops and budget escalations themselves within the table's bound

`driver-settles-stops` · Task · feature, shed, board, batten, shuttle, webster, arbiter, loom, stencil

Decision record: [Sketch](decision-driver-settles-stops.md)

Depends on: [`stop-settlement-table`](design-stop-settlement-table.md)

## The driver settles board claim refusals, failed launches, bounce budgets and gate attempts itself within the table's bound

Filed from the Sketch `self-healing-runs`.

Takes: D4's driver rows, D2's driver steps and D3's bound check, over stop-halt-count's count function and stop record and stop-settlement-table's bounds. Raise sites: the claim push in `pushOwnerCommit` in `internal/boardengine/ownerset.go`, reached from `claimBoardEntry` in `internal/battencli/owner.go`; `shuttleengine.ErrNotStarted` and `shedtransient.TransientAgentStart`; the session timeout reason in `internal/shedadapters/multillm.go`; the fixer fork in `internal/websterengine`; `strandGoneError` in `internal/shuttleengine/run.go`; and the gate-attempts Stuck from `gateFailedReason` in `internal/shedadapters/gateattempts.go`.
Delivers: New rows in `decideDrive` in `internal/sheddriver/driverules.go`: a board claim refusal gets a board sync and a re-step; a launch never ready, a session timeout, a failed fixer fork and a lost strand get one fresh start of the row's producer; a bounce budget exhausted gets one resume, matching the `shedengine.ReasonBounceBudgetExhausted` prefix; a gate-attempts Stuck gets a re-step with a fresh producer session. Each class but the bounce budget's is raised as a named kind or field where the stop starts. Each row takes its bound from the table, a settle within the bound sends no notice and leaves a settle record, and a stop past it is handed back with its verdict. The tackler keeps every stop no row names.
Exports: The named kinds for the claim refusal, the session timeout, the fixer fork failure, the lost strand and the gate-attempts Stuck.

The part changes these packages:

- internal/sheddriver (about 650 changed lines)
- internal/boardengine (about 70 changed lines)
- internal/battencli (about 30 changed lines)
- internal/shuttleengine (about 130 changed lines)
- internal/shedadapters (about 140 changed lines)
- internal/shedtransient (about 50 changed lines)
- internal/websterengine (about 80 changed lines)

## The arbiter takes budget escalations, and nothing lands a round with an open blocking finding unattended

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

