# Fixer report — discussion, round 1

Every finding's premise held against the code: `Runner.Inject` in `internal/shuttleengine/run.go`, `readinessReasonNoInputBox` in `readiness.go`, `handleHubEntry` and `settledWithoutInterrupt` in `internal/orchengine/pause.go`, `sessionReadiness` in `readiness.go` and `interruptSession` in `internal/pauseengine/pause.go`.
All changes are in `_lyx/discussion/decision-record.md`; `support-log.md` needed none.

## Changes

- F1 (fixed as proposed, Inject-refuses branch): D7 names `Runner.Inject` as the one raw key path exempt from the guard but forbidden the interrupt; Inject refuses an input list equal to the engine's interrupt sequence, sends nothing and returns an error naming `lyx shuttle interrupt`, so `creel` reaches its Esc through the guarded `Interrupt` only.
  D7 gains a bound that an input list holding Escape among other keys still plays.
  New Acceptance criterion pins the refusal, and notes that the existing Inject refusal tests in `run_inject_test.go` and `run_test.go` that pass a lone `Escape` to reach another refusal switch to a non-interrupt input; the happy-path test plays Escape with text, so it is unaffected.
- F2 (fixed as proposed): D4's pause-engine bullet says the state predicate decides the second-Esc attempt after the first settle, a guarded `ErrNoRunningTurn` on the second Esc ends as interrupted, and after a second Esc the engine waits the settle again through `Deps.Sleep` and classifies by the pane through the guard's running-turn capability, via the `Pair` seam.
  D5's `interrupted` and `still_busy` name that reading, and the Acceptance criterion on the second Esc says the classification follows the settle after it.
- F3 (fixed as proposed): D6's example and the pre-change-entry criterion quote the guard's reason, "no input box is on screen; a menu or picker may be open".
- F4 (fixed as proposed): D4's orch-watcher bullet orders the remembered first-Esc time before `settledWithoutInterrupt`, marks the entry handled only after the second Esc or the settled idle re-probe, and states the restart bound.
- F5 (fixed as proposed): D3 says the orch watcher logs a guarded refusal once per hub entry at Info naming the orch strand, never Warn; the running-turn-needle Open risk names that Info line as where the orch's case shows; the orch-watcher criterion expects the Info line and no Warn.
- F6 (fixed as proposed): D1 now claims a shared state rule, not a shared reading, and names how the readings differ; a new Open risks item covers the unmatched-turn-start case landing in `already_idle` and the released-hold case landing in `no_running_turn`.

## Disputed

None.

## Deferred

None.
