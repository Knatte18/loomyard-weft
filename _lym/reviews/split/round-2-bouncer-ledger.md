---
round: 2
ledger:
  - key: hold-setters-no-production-caller
    rounds: [1]
    status: resolved
  - key: handover-mark-drop-unbuilt
    rounds: [1]
    status: resolved
  - key: root-beam-heads-chain
    rounds: [1]
    status: resolved
  - key: adopt-single-bolt-commit-setter
    rounds: [1]
    status: resolved
  - key: fix-tries-reader-not-kept
    rounds: [1]
    status: resolved
  - key: launcher-removal-misses-catchup
    rounds: [1]
    status: resolved
  - key: criteria-built-by-later-part
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: fix-tries-reader-route
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1's six findings were not re-raised in round 2, and the round 2 review's Rejected section clears the hold setters, the hand-over mark drop and the chain.
Round 2 raised two design findings.
F1 (BLOCKING) says `transition-push-takeover` carries three criteria that later parts build; the fix moves each bullet to the part that delivers the behavior.
F2 (MEDIUM) says the upgrade's first read has no fixed route to the fix-tries reader, and the two parts' bodies disagree on who exposes it.
