---
verdict: CONTINUE
rationale: "Round 3 carries one BLOCKING finding (F1, the reconcile's mutation record never reaching the mutations array of orch start, cleanup or wake); no finding is design-class and none sits on a key open in an earlier round, so the rule gives CONTINUE, not CIRCLING."
---

The latest review has six findings: F1 BLOCKING, F2, F3 and F4 MEDIUM, F5 and F6 LOW.
The decision rule is `CONTINUE` while the latest round carries any BLOCKING finding, so F1 alone decides it.
No finding is design-class, so no gating-class finding sits on a key open in an earlier round.
`CIRCLING` is therefore not offered.

No finding is relabelled, and the review carries no `## Focus departures` section.

Mapping to ledger keys:

- F1 is the new key `reconcile-mutations-not-in-envelopes`.
  It resembles the round-2 key `pair-removals-bypass-mutation-record`, but that key was about the removals filling a record at all, and card 17 now does.
  F1 is about the record reaching the verbs' envelopes, so the earlier key is carried as resolved.
- F2 is `records-push-verb-worktree`, F3 is `way-forward-line-unasserted-at-print-hops`, F4 is `prime-flag-and-class-rows-unreached`, F5 is `two-sided-push-failure-line-duplicated`, F6 is `dirty-line-no-bolt-form`.
- `failedpushwayforward-dead-ends` and `transport-class-no-backoff` are not re-raised and carry as resolved.
