# Fixer report — discussion round 3

## Changes

- F1 (MEDIUM, fixed as proposed, extended): premise confirmed, `checkChild` in `internal/battenshed/innerrun.go` ends the wait on any state change, so a capped retry is re-entered.
  D5 now names `started` and `goto-done` as the unfinished states the suppression covers, adds a `capped` marker state written when the cap's notice is queued, measures the cap from the `started` write, and says `resumed` and `capped` each end the run's one retry.
  Beyond the proposed fix, D5 also names the reviewer's unhandled combinations: `started` with the child paused resumes it, and an unfinished marker in any other state counts as the retry spent.
  The Bound says a capped retry counts as the one, and the cap criterion adds that a restart does not reset the cap and that a second failure after the cap's notice and an orch resume gets the blocked notice and no goto.
- F2 (LOW, fixed as proposed): premise confirmed, `internal/fabricengine/pushretry.go` retries "(failed)" only around Add's branch pushes, and `rebaseRetryTriggers` in `internal/gitrepo/push.go` catches it elsewhere.
  Scope In now names the "(failed)" widening, and Scope Out says that besides D6's push refusals only a Publish verify failure is automated.
- F3 (LOW, fixed as proposed, first option): premise confirmed, `PRGate.Call` in `internal/landingshed/prgate.go` returns Done unreviewed for a parent outside `require_pr_to_base`, read at gate time.
  D2 now rests trust on the listed branch's remote receiving code only through PR-Gate-reviewed landings or the operator, states that the list is checked when the Land row runs, names the list-edit gap, and bounds the deploy to the branch's remote head.
  The second option, reading the child's PR-Gate outcome, was not taken: PR-Gate's Done does not record whether it reviewed or skipped, so it would need a new record field for a gap only the orch's own config edit opens.
- `support-log.md`'s Review rounds gains the round 3 entry.

## Disputed

None.

## Deferred

None.
