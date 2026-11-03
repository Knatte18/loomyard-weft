---
round: 2
ledger:
  - key: scope-replacement-loses-capture-skip-list
    rounds: [1]
    status: resolved
  - key: anchored-start-dir-not-pinned-at-wiring
    rounds: [1]
    status: resolved
  - key: failed-capture-removal-leaves-scope-replaced
    rounds: [1]
    status: resolved
  - key: gateslot-holder-worktree-became-anchor
    rounds: [1]
    status: resolved
  - key: non-breakaway-start-error-offered-defer
    rounds: [1]
    status: resolved
---

Round 1 raised five findings.
Round 2's fresh review found none and rejected the matching focus items: the capture skip-list comparison, the anchored start directory pinned through every call site, the scope write ordering around capture removal, the holder's documented worktree field, and the capture halt limited to a child that never recorded its start.
All five keys are resolved.
