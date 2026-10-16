---
round: 2
ledger:
  - key: held-measures-elapsed-on-every-call
    rounds: [1]
    status: resolved
  - key: old-watcher-types-conditioned-notice
    rounds: [1]
    status: resolved
  - key: guard-branches-without-test-rows
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card-12-grep-claim-misses-error-prefixes
    rounds: [1]
    status: resolved
  - key: attach-help-text-stale-on-unix
    rounds: [1]
    status: resolved
  - key: revive-caller-refusal-documented-twice
    rounds: [1]
    status: resolved
  - key: callout-timeout-package-var-in-selfreport
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: held-page-row-bypasses-pause-gate
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: conditioned-send-retrylanded-false-landed
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: orch-comments-claim-state-decides-nothing
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: held-page-row-thin-wallclock-margin
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none re-raised in round 2 except in one case, so five are resolved.
`guard-branches-without-test-rows` recurs as F5, card 6's state re-read error branch that no row reaches.

Round 2 raised six findings.
F1 is BLOCKING: card 3's `callTimeout` var breaks `PATTERN-test-speed`, and its fix introduces a `listOpenIssues(perRequest)` function, so it needs a fresh review.
F2 and F6 concern card 3's held-page row: it never goes through the pause gate, and its timing margin is thin.
F3 is a card 4 design flaw: `typeVerified` carries `retryLanded`, so a conditioned send can report a line it never typed as landed.
F4 is a card 5 comment-consistency gap in orchengine.
