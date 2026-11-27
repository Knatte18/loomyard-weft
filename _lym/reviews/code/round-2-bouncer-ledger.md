---
round: 2
ledger:
  - key: redfirst-refusals-untested
    rounds: [1]
    status: resolved
  - key: merge-danger-omits-older-binary-reports
    rounds: [1]
    status: resolved
  - key: stored-testcover-data-left-on-disk
    rounds: [1]
    status: resolved
  - key: test-speed-pattern-misdescribes-testtiming
    rounds: [1]
    status: resolved
  - key: gateslot-doc-nontier-holder-list-empty
    rounds: [1]
    status: resolved
  - key: rowgate-comment-stale-rerun-mark
    rounds: [1]
    status: resolved
  - key: retired-gate-key-load-unpinned
    rounds: [1]
    status: resolved
---

Round 1 raised every key above; `redfirst-refusals-untested` was the one gating key.
Round 2 reports each resolved and raises no new finding.
