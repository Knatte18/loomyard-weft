---
verdict: CIRCLING
rationale: "Gating design findings F1 (BLOCKING, D7 merge way forward) and F3 (MEDIUM, recover-batch named for an uncheckable failed batch) sit on keys open in rounds 1 and 2 and are still open in round 3; F4 and F5 are further open gating MEDIUM design findings."
---

Round 3 carries one BLOCKING finding (F1) and four design-class findings at MEDIUM or worse (F1, F3, F4, F5).

Two gating findings map to keys the facts list as open in both earlier rounds:

- F1 maps to `d7-failed-remote-update-and-diverged-state` (rounds 1, 2).
  Round 3 escalates it to BLOCKING: the plain-merge way forward ends in a re-run that discards what it merged.
- F3 maps to `d6-recover-batch-refuses-uncheckable-failed-batch` (rounds 1, 2).
  The batch-failed reason and D6's rebaseline still name `recover-batch` for a batch it refuses.

That makes the verdict `CIRCLING`, not `CONTINUE`.

Other open gating findings:

- F4 maps to `d6-dead-batch-no-fork-premise` (round 2): the restamp of a dead batch states no bound on a live strand.
- F5 maps to `d10-removed-assertion-unbounded` (round 2): D10 describes a check the code does not perform.

F2 (decision, MEDIUM) maps to `d7-remote-update-order-before-local-checks` and is non-gating by class.

No focus departures section exists in the round 3 review, so there is nothing to ratify or reject.
No finding is relabelled.

Resolved this round: `d3-relaxation-let-through-unstated` (review's focus verdict: not a defect), `d6-acceptance-omits-dead`, `d7-remote-tip-ancestor-of-reset`, `d1-deriveedges-mirror-cross-batch`, `open-risks-repeats-d2-bound`.
