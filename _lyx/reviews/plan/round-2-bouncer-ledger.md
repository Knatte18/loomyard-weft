---
round: 2
ledger:
  - key: gateslot-watchdog-line-bundled-in-gatecli-card
    rounds: [1]
    status: resolved
  - key: rerun-skip-no-stated-bound
    rounds: [1]
    status: resolved
  - key: card2-misses-comments-made-wrong
    rounds: [1]
    status: resolved
  - key: card4-replace-all-contradicts-rename-target
    rounds: [1]
    status: resolved
  - key: timed-out-step-not-reported-as-not-rerun
    rounds: [1]
    status: resolved
  - key: enclosing-variable-bound-overstates-ignored
    rounds: [1]
    status: resolved
  - key: sampler-failure-branch-untested
    rounds: [1]
    status: resolved
  - key: negative-key-refusal-wording-mismatch
    rounds: [1]
    status: resolved
  - key: finalize-verify-failed-no-refusal-row
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
  - key: loomshed-round-record-rewrite-drops-unknown-keys
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: watchdog-stderr-writer-no-route-into-runtest
    rounds: [2]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised eight findings, none of which round 2 raised again; all are resolved.
Round 2 raised three new findings: F1 (card 14, Finalize verify refusal row, BLOCKING), F2 (card 7, loomshed round record rewrite, MEDIUM) and F3 (card 11, watchdog stderr route, LOW).
None is `design`, so none is gating by class, but F1 is BLOCKING and forces CONTINUE.
