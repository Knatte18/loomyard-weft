# Fixer report — discussion, round 1

Every premise was checked against the code: `internal/clihelp/annotations.go` (the audience annotation only filters the index), `internal/landingshed/parentguard.go` and `internal/loomcli/landingdeps.go` (the `PauseRequested` seam reads `pause_requested`), `internal/parentlock/acquire.go` (`ErrPaused`), `internal/parentlock/idle.go` (idle release), and the support log's Rejected alternatives entry on `lyx shuttle interrupt`.
All eleven hold.

## Changes

- F1 — fixed differently in one respect: D1 now bounds the callers. The orch and an `EXTERNAL` caller may run either verb; an agent (a shuttle-run strand of a pair) may only pause or resume its own pair's slug, and a hub-wide pause from it refuses, naming the way forward of asking the orch or the operator. The proposed fix bounded only `lyx pause`. The same bound now also covers a hub-wide `lyx resume` and either verb naming another pair's slug, because otherwise an agent could lift an operator's hub pause, the same unbounded hub-wide reach the finding names. Added to the Constraints refusal list and as a Caller acceptance criterion, and the Open risks bullet now names an operator- or orch-left pause.
- F2 — fixed as proposed: D4 says a covered run is never granted the parent lock while covered, so the exception reaches only a run that held the lock before the entry was written. It lets through that run's remote calls of its current landing row, which D3 step 4 and `lyx pause --status` name. Added the acceptance criterion that a covered waiter is not granted the lock until the resume. D4 also notes that a `--after` naming the waiting row still ends the wait (F6), so the two decisions agree.
- F3 — fixed as proposed: D5's idle-release bullet now states the behavior (no idle release for a covered holder; queued runs wait; the report and status name held branches and queued runs). D4's rationale clause now says what the exception prevents and that other runs still wait behind a covered holder. D3 step 4 lists held branches and queued runs. Added the queued-behind-a-covered-holder acceptance criterion.
- F4 — fixed as proposed: D3 step 1 records the grace end in the entry. D7's watcher interrupts the orch only once the grace has ended, or earlier once no orch network marker is open, the D3 step 2 rule. D3 step 4 reports the orch's case (delegated to a live watcher, or not interrupted because none is live). Extended the orch acceptance criterion.
- F5 — fixed as proposed: D7 queues the resume line only when the entry owns the orch, and an orch idle at pause time or the caller gets none. Extended the orch acceptance criterion.
- F6 — fixed as proposed: in D8, landingshed's seam reads `pause_after` naming the waiting row, the "parent lock wait paused" row stays with its trigger reworded, and D8's Bound names this as the one remaining mid-row halt. Added the Boundary pause acceptance criterion.
- F7 — fixed as proposed: D10 gains a Bound line (only runs whose record names the killed step's run directory; never the driver, the orch or another step's runs; an untied run is left and named in the halt note). Extended the watchdog-kill acceptance criterion.
- F8 — fixed as proposed: D9's continue line goes through shuttle's guarded paste-submit, and a refusal sends nothing and holds and notifies the stall. D9's Bound says the same.
- F9 — fixed as proposed: the Goal now says the bare loom and webster boundary pause goes.
- F10 — fixed as proposed: deleted the `lyx shuttle interrupt` line from D4. The Rejected alternatives entry stays.
- F11 — fixed as proposed: the Question ledger entry is resolved and points at D9.

## Disputed

None.

## Deferred

None.
