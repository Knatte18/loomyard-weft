---
round: 2
ledger:
  - key: verify-skips-tmux-loomcli-webstercli
    rounds: [1]
    status: resolved
  - key: reed-unless-name-docs-stale
    rounds: [1]
    status: resolved
  - key: loom-status-waiting-docs-stale
    rounds: [1]
    status: resolved
  - key: verify-timeout-webster-landing-docs
    rounds: [1]
    status: resolved
  - key: reed-resume-glue-untested
    rounds: [1]
    status: resolved
  - key: reed-resume-unlicensed-parent-condition
    rounds: [1]
    status: resolved
  - key: verifygate-test-false-premise
    rounds: [1]
    status: resolved
  - key: shuttle-wait-duplicate-empty-row
    rounds: [1]
    status: resolved
  - key: batten-respawn-retiring-driver-unstated
    rounds: [1]
    status: resolved
  - key: webster-verify-timeout-spec-overview-stale
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: verify-note-site-clause-contradicts-decision
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: batten-spawn-confirmation-docs-stale
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: reed-drop-rule-not-hidden-condition
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised nine findings, one of them `design`; none of the nine reappeared in round 2, so all are resolved.
Round 2 keys map to F1 through F4 in that order: stale webster spec and overview after card 9, the site clause in card 10's note, stale batten spawn-confirmation docs after card 16, and card 5's extra not-hidden drop condition.
No round-2 finding is `design` or BLOCKING, so the loop converged.
