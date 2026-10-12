---
round: 1
ledger:
  - key: publish-comments-stale-merge-in
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised one NIT, F1, mapped to `publish-comments-stale-merge-in`: two `Publish` comments still mention a merge-in that Publish no longer runs.
Nothing else was raised, and there was no earlier ledger to carry forward.
