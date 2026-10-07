---
round: 1
ledger:
  - key: verify-skips-tmux-loomcli-webstercli
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: reed-unless-name-docs-stale
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: loom-status-waiting-docs-stale
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: verify-timeout-webster-landing-docs
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: reed-resume-glue-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: reed-resume-unlicensed-parent-condition
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: verifygate-test-false-premise
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: shuttle-wait-duplicate-empty-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: batten-respawn-retiring-driver-unstated
    rounds: [1]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised nine findings: five MEDIUM and four LOW.
Keys map to F1 through F9 in that order.
F1 is the only `design` finding and gates the round; F2 to F5 are stale or missing doc edits and an untested path.
F6 to F9 are small fidelity, premise, test-economy and bound-statement issues.
