---
round: 2
ledger:
  - key: conflict-stopper-misses-numbered-strand
    rounds: [1]
    status: resolved
  - key: failed-reason-names-two-ways-forward
    rounds: [1]
    status: resolved
  - key: refusal-omits-operator-unnamed-shell
    rounds: [1]
    status: resolved
  - key: abort-reach-unbounded-cards-5-6
    rounds: [1]
    status: resolved
  - key: decision-add-check-states-no-bound
    rounds: [1]
    status: resolved
  - key: midmerge-shas-side-unstated
    rounds: [1]
    status: resolved
  - key: newloomcli-seam-outside-seam-test
    rounds: [1]
    status: resolved
  - key: probegated-subcase-duplicates-attach-row
    rounds: [1]
    status: resolved
  - key: abort-failure-reason-has-no-test-row
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
  - key: parent-branch-read-error-has-no-test-row
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1's eight findings were all fixed in the cards, and round 2's review confirmed none regressed, so all eight are resolved.
Round 2 raised two new test-coverage gaps, both introduced by round 1's fixes: F1 (BLOCKING) is the abort-failure Stuck reason in card 5, F2 (LOW) is the parent-branch read error in card 6's guard.
