---
round: 3
ledger:
  - key: held-measures-elapsed-on-every-call
    rounds: [1]
    status: resolved
  - key: old-watcher-types-conditioned-notice
    rounds: [1]
    status: resolved
  - key: guard-branches-without-test-rows
    rounds: [1]
    status: resolved
  - key: card-12-grep-claim-misses-error-prefixes
    rounds: [1]
    status: resolved
  - key: attach-help-text-stale-on-unix
    rounds: [1]
    status: resolved
  - key: revive-caller-refusal-documented-twice
    rounds: [1]
    status: resolved
  - key: callTimeout-package-var-breaks-injectable-time
    rounds: [2]
    status: resolved
  - key: held-page-row-bypasses-pause-gate
    rounds: [2]
    status: resolved
  - key: typeverified-keeps-retrylanded-check
    rounds: [2]
    status: resolved
  - key: stale-session-state-comments-after-conditioned-notice
    rounds: [2]
    status: resolved
  - key: restartsafe-state-reread-branch-unreachable
    rounds: [2]
    status: resolved
  - key: held-page-row-20ms-margin
    rounds: [2]
    status: resolved
  - key: paste-row-unreachable-apierr-parser
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: held-page-row-page1-30ms-slack
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1's six findings and round 2's six findings do not reappear in round 3, so all are resolved.

Round 3 raised two new MEDIUM findings.
F1 is card 4's paste-path row, which cannot reach the paste path because the fake's session-signal parser drops `APIERR`.
F2 is card 3's held-page row, which leaves page 1 thin timing slack.
Both fixes are local numeric or fake-parser corrections that the finding text already fixes.
