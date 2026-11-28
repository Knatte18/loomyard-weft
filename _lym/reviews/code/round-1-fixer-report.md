---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The remembered origin-wait stop is kept in daemon memory (originStops, with a counter of successful catch-ups) instead of a StartState field. A persisted mark would outlive the dobby and refuse a task on its first blip after a restart, and would leave a start record where the existing origin-wait test asserts none. A re-exec that loses it costs one more owed catch-up before the report."
  - id: F2
    disposition: fixed-differently
    reason: "The seam is BeamCatchUpWaits and lists beams only; the prime's own wait is still read from its record, so no nil-seam fallback replaces that read. The wiring also leaves out a beam whose dobby is not live, since nothing retries or clears a stopped dobby's record and listing it would hold back the all-clear indefinitely."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

## Changes

- F1 (`internal/dobbyengine/starts.go`, `loop.go`, `catchupwait.go`): an origin-wait first pass is a refused start when the dobby has no catch-up,
  or when a catch-up succeeded since an earlier such stop of the same entry.
- F2 (`internal/dobbyengine/pushsummary.go`, `internal/beamcli/dobbydeps.go`): the prime's summary lists each live beam dobby's catch-up waiting on origin.
  A tier 1 test pins the wired read.
- F3 (`internal/battencli/drive.go`, `internal/dobbyengine/geometry.go`, `passes.go`): only an unreachable origin exits `PassExitOriginWait`;
  every other catch-up failure exits `PassExitCatchUpRefused`.
- F4, F8: the package doc names the catch-up line, and the heading reads "waiting on the network".
- F5, F6, F7: added the missing subtests.
- `_lym/landing/summary.md`: corrected for F1, F2, F3 and the self-check fixes.
- Gate finding on comment lines: rewrote the flagged comments in `internal/battencli/catchup.go`, `internal/battencli/run_test.go`,
  `internal/dobbyengine/geometry.go` and `internal/dobbyengine/passes.go` as one sentence or clause per line.
  Only comments changed.

Gate: `lym gate test` over `internal/dobbyengine`, `internal/battencli` and `internal/beamcli` passed in tier 0 after the comment repair
(0 failed, 77 skipped).
The tier 1 push-summary wiring tests of `internal/beamcli` passed before the self-check fixes, which did not touch that package.
No other tier 1 or higher test was run.

## Self-check

- fix-code-correctness, MEDIUM, a live dobby's wait is never retried after a re-exec: fixed. A re-exec owes the catch-up while the record holds a wait.
- fix-code-correctness, MEDIUM (related part of the same finding), a restarted dobby keeps the old `Since`: fixed. A fresh start drops the recorded wait.
- fix-proposal, MEDIUM, F2 reports a wait nothing clears after a re-exec: fixed, with the line above.
- fix-proposal, MEDIUM, F1 forgets a remembered stop too late: fixed. Any other exit of the first pass forgets it.
- fix-proposal, MEDIUM, F1 reads a second outage as the start's own failure: deferred, below.
- fix-consistency, MEDIUM, the docs say a live dobby's wait is retried when a re-exec loses it: fixed, with the re-exec change.
- fix-consistency, LOW, the catch-up section of the package doc on when the catch-up is owed: fixed.
- fix-consistency, LOW, the `run_test.go` header does not name the refused-catch-up exit: fixed.
- fix-consistency, LOW, the all-clear still speaks only of runs: fixed. It reads "nothing waits on the network".
- fix-affected-set, MEDIUM, a wait nothing retries after a re-exec: fixed, with the re-exec change.
- fix-affected-set, same finding's smaller case, a beam dobby under an adopt hold lists its old wait: not changed (premise holds, no change made).
  The catch-up there is still unfinished, so the record states it correctly, and it is retried once the hold lifts.

## Disputed

## Deferred

- fix-proposal, second part of its F1 finding: on a flapping link, origin can drop again during the retried first pass.
  That second stop is then reported at once, with a backoff, although the goal reports only past a bound.
  The exit code cannot tell this from a failure of the task's own, which F1 requires to be bounded.
  The review's fix reports at the second stop; a higher count (as the three of a failing pass) or a time bound would tolerate more flapping and report a real fault later.
  Which bound to use is a tradeoff for the run's parent. The code keeps the review's second stop.

## Rulings
