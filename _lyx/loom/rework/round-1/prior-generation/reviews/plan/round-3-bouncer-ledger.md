---
round: 3
ledger:
  - key: chair-include-breaks-single-agent-composer
    rounds: [1]
    status: resolved
  - key: seat-runner-gate-once-own-card
    rounds: [1]
    status: resolved
  - key: seat-runner-gate-once-double-eval-and-blast
    rounds: [1]
    status: resolved
  - key: advisors-mapping-refusal-way-forward
    rounds: [1]
    status: resolved
  - key: d4-deviations-unrecorded
    rounds: [1]
    status: resolved
  - key: seatengine-resigns-vs-additive-constraint
    rounds: [1]
    status: resolved
  - key: stale-discussionspec-references
    rounds: [1]
    status: resolved
  - key: reconcile-test-duplicates-mechanism
    rounds: [1]
    status: resolved
  - key: card7-discussiontable-seam-red-until-card8
    rounds: [2]
    status: resolved
  - key: card8-recipeengines-test-not-targeted
    rounds: [2]
    status: resolved
  - key: gate-and-brief-acceptance-criterion-uncarried
    rounds: [2]
    status: resolved
  - key: chair-stencil-single-agent-framing
    rounds: [2]
    status: resolved
  - key: card8-false-branch-subtest-redundant
    rounds: [2]
    status: resolved
  - key: card7-call-test-conflicts-with-gate-once-fake
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: discussion-seats-row-bypasses-gate-quiescence-guard
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
---

Rounds 1 and 2 raised thirteen findings; round 3's review raises none of them again, so all are resolved.
Round 3 raised two new MEDIUM findings.
F1: card 7's `TestDiscussionSeatsEntry_Call` scripts done chair results without a `GateOutcome`, so card 6's gate-once fake fails the commit row and opens a round in the resume row.
F2: card 8's engine substitution to `DiscussionSeats` runs after the parse, so the gated row escapes the gate-quiescence guard, and card 8's Intent states no bound.
F2 is gating-class but first raised this round, so Go carries it into `## Open risks`.
