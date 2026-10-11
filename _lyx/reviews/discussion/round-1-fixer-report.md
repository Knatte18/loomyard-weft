# Fixer report — discussion round 1

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  Premise checked: `appendHistory` in `internal/shedengine/run.go` records nothing for a producer error with no outcome.
  D1 now names the stop key once (`current_producer`, `history_length`, `state`), says the re-launch writes an applied mark into the attempt directory when it consumes a `resume` attempt, and says a re-run returns an unapplied verdict and refuses once an applied `resume` attempt holds the key.
  D7's same-stop rule now refers to D1's stop key and the applied mark, and names the error-stop case.
  D6's wake sentence says "unapplied attempt for the current stop key".
  Criteria: the re-run criterion and the wake criterion say "unapplied", and a new criterion pins a producer error recurring after an applied `resume` being handed on without a second spawn or re-launch.
  Of the review's two options for who writes the mark, the re-launch writes it, since it does so in Go at the moment the verdict is applied.
- **F2 (MEDIUM), fixed differently in one part.**
  Premise checked: `goto` leaves the run paused, and `pause` sets `pause_requested`, which the machine consumes at its next boundary.
  D6's `resume` bullet now has the re-launch, in Go, check that the run is in the state the arbiter's records left before it un-pauses: `pause_requested` unset, and either no recorded `goto` with the stop key unchanged, or one recorded `goto` whose entry is the only history added and whose row the verdict names, with the run `paused`.
  Any other state is handed on with the verdict path.
  The second Open risks bullet now states this bound.
  Not done as proposed: extending the orch-goto criterion while keeping its "re-launches over the moved state" outcome.
  The bound the fix key itself proposes (the last history entry must be a goto the attempt recorded) refuses an orch `goto`, so keeping that criterion would contradict D6; the criterion now says an orch `goto` or `pause` during a live arbiter is handed on, not un-paused or re-launched.
- **F3 (MEDIUM), fixed as proposed.**
  D2 gains a Bound line: a verb run at a `producer`, `bootstrap` or reinvoke `interrupted` error without a fork escalation skips the fork's diagnosis and the repair cap, bounded by the spawn budget, the applied-attempt refusal (D1, D7) and the arbiter's lack of repair verbs (D3).
- **F4 (MEDIUM), fixed as proposed (first option).**
  Premise checked: both `MarkWatched` call sites in `internal/battenshed` (`seamchild.go`, `innerrun.go`) log and carry on when the marker cannot be written.
  An absent marker is what both an unwatched run and a failed `MarkWatched` leave, so "cannot positively read the run as unwatched" holds for every run; D2 now refuses the target-naming Publish halt in every run and drops the watched-marker read from the verb.
  The Scope Out bullet, the first criterion and the Auto-mode assumptions line follow.
- **F5 (LOW), fixed as proposed.** Scope's `(D14)` is now `(D13)`.
- **F6 (LOW), fixed as proposed.**
  D8 refuses a negative `arbiter_timeout_min` naming the key, as D7 does for the budget; the absent-keys criterion says so.
- **F7 (LOW), fixed as proposed.**
  The batten criterion now says a marker whose arbiter shuttle run is dead, and a missing marker, hold nothing and the notice goes as today.

`support-log.md`'s Review rounds section records round 1.

## Disputed

None.

## Deferred

None.
