---
round: 1
ledger:
  - key: readmailbox-stale-state-on-ack-failure
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: retryfreshlaunch-loses-withdrawn-resume-line
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: creelseam-mapping-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: goclient-timeout-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: battenshed-stale-queue-comments
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings, none seen before.
Three are gating: `readMailbox` saving the pre-handler state after a failed read acknowledgement, `retryFreshLaunch` losing a withdrawn resume line when the mark clear fails, and the untested `creelSeam` mapping.
The LOW and NIT findings concern a missing timeout pin on `GoClient` and stale queue wording in `battenshed` comments.
