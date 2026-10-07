# Round 1 fixer report

All seven findings fixed in `_lyx/discussion/decision-record.md`; `_lyx/discussion/support-log.md` was brought in line.
`lyx loom validate-discussion` exits ok on the result.

## Changed

- F1: Decision 5's bound now says both verbs push unconditionally, so in the prime they publish unpushed parent-branch commits even when the call committed nothing, including via the `sync` that `lyx config` runs; the bound is a plain rebase-free push, and `lyx fabric commit` already publishes the same commits when it lands something.
  Auto-mode assumption 3 now names only Batten's per-transition seam and points at Decision 5.
- F2: Decision 3's bound is now a bounded, injectable wait on the status side, which warns on expiry so the run continues; the detached child keeps the blocking acquire; the false "one push's duration" claim is replaced by the real holder cost.
  The Open risks lock entry is rewritten to the residual risk, and an acceptance criterion covers a status push against a held lock.
- F3: Decision 1 now returns a single failed side's error as it is and `errors.Join` of both with the side named when both fail, so `errors.Is` still finds a rejection.
  Decision 2 has the warning and envelope carry that text and drops the claim that the records were not pushed.
  An acceptance criterion covers the different-error-kinds case.
- F4: Decision 5 now has `push` call the new entry, so a rejection is an error envelope, and states that `PushCoalesced`'s `pull --rebase` recovery no longer applies to `lyx fabric push`.
  An acceptance criterion covers `lyx fabric push` against a real divergence.
- F5: Constraints gain `PATTERN-refusal-way-forward`, naming the `commit-records` message and its `contracts/specs/refusal-spec.md` row.
- F6: The `PATTERN-push-both-sides` entry text now carries the prime exemption.
- F7: Assumption 1 is cut to the unverified-cause fact, and the first Notes bullet (the call-site list) is deleted.
- Support log: three rejected alternatives and ledger entries Q7 to Q10 record the new picks.

## Deferred

Nothing deferred.
Two picks are the agent's own in autonomous mode and are recorded in the ledger as auto-picks: the bounded status-side wait (F2) and accepting the prime's parent-branch exposure through `push` and `sync` (F1).
