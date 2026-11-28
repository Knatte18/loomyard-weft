# The driver settles board claim refusals, failed launches, bounce budgets and gate attempts itself within the table's bound

`driver-settles-stops` · Task · feature, shed, board, batten, shuttle, webster · draft

Decision record: [Sketch](decision-driver-settles-stops.md)

Depends on: [`stop-halt-count`](design-stop-halt-count.md), [`bouncer-stalemate-uncommitted`](design-bouncer-stalemate-uncommitted.md), [`missing-origin-unreachable`](design-missing-origin-unreachable.md)

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
