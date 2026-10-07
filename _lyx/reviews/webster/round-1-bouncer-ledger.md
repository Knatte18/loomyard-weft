---
round: 1
ledger:
  - key: comment-line-breaks-fixed-column
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: new-tests-omit-t-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: verify-gate-stuck-reason-no-timeout-case
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: revive-once-marker-ignores-history-episode
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: strandnamed-test-duplicates-public-test
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: overview-loom-bullet-two-sentences-one-line
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings, none recurring.
Two are BLOCKING and consistency class: changed Go comments that break the semantic line-break rule (F1), and new tests that omit `t.Parallel` or leave a logger-output swap unnamed (F2).
One is a gating `design` finding at MEDIUM: a timed-out verify gate that exhausts its attempts reports no identities (F3).
Three are LOW: the revive once-marker outliving a halt episode (F4), a redundant `TestStrandNamed` (F5), and a two-sentence line in `docs/overview.md` (F6).
