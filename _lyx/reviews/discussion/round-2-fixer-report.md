# Fixer report — discussion round 2

## Changes

- F1 (BLOCKING), fixed as proposed, with the row-neutral told clause chosen.
  The premise was confirmed in the code: `internal/shedengine/reroute.go` ends the cap reason with the told clause, and `loomRerouteCapWayForward` in `internal/loomcli/wiring.go` names only the parent branch.
  D3 now says the cap halt is reachable at the Webster row and makes `loomRerouteCapWayForward` row-neutral (resume once the cause the reason quotes is cleared, with the parent-branch case and the Webster plan-conflict hand edit both spelled out).
  The adapter's reroute reason now names the plan-conflict way forward, so the cap reason quotes it as well.
  Scope's loom bullet names the clause change.
  D5 adds the loom refusal-spec row for a Webster reroute-cap halt.
  Acceptance criteria add a bullet saying a capped Webster reroute names that way forward and no parent branch.
  Notes for the plan writer point at `reroute.go` and `wiring.go`.
- F2 (MEDIUM), fixed by the second option the fix key offers: the write stays in the adapter.
  D3 states that a capped conflict is recorded as routed, so resuming the cap halt without an edit halts as a repeated conflict.
  The repeated-conflict reason now says the conflict "recurred after a reroute of it was asked for" and does not claim a review took place.
  The Bound reads "at most one plan review ..., and none when shed caps its reroute" and notes that Webster is Done only at the end of its run.
  Open risks gains the capped case.
  Acceptance criteria add two bullets: the reasons never claim a review, and a cap resume with no plan edit halts as repeated.
  The support log's Rejected alternatives record why the write was not moved to `Plan-Burler-Review` (`BurlerReviewProducer` serves every review segment).
- F3 (LOW), fixed as proposed (first option).
  D2 now puts the removal at the verb's first step, before batch-number parsing, plan parsing, lock acquisition, state load and batch derivation.
  This order matches `internal/webstercli/beginbatch.go`, where `ParsePlan`, `AcquireStateMutation`, `LoadState` and `executionBatches` all come before `BeginBatch`.
  D2's Rationale and the acceptance bullet for non-conflict refusals now name the earlier refusals.
  The support log's Q2 ledger entry is updated to match.
- F4 (NIT), fixed as proposed: a blank line now closes D3's bullet list.
  The new plan-conflict way-forward paragraph starts after it.

## Disputed

None.

## Deferred

None.
