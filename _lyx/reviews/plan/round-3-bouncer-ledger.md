---
round: 3
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
    status: resolved
  - key: parent-branch-read-error-has-no-test-row
    rounds: [2]
    status: resolved
  - key: stop-seam-empty-guid-becomes-stuck
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
  - key: parent-branch-read-precedes-producer-check
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Rounds 1 and 2 findings are all resolved; round 3 confirmed the AbortMerge and probe-error test rows exist.
Round 3 raised two new keys: F1 (BLOCKING, design) is the empty-guid Stuck reason when `conflictSessionStopper` cannot read the strand table, and F2 (LOW, consistency) is the parent-branch read in `refuseOverUnfinishedMerge` happening before the producer and verb check.
