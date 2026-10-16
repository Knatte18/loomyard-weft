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
  - key: calltimeout-package-level-var
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
  - key: typeverified-keeps-retrylanded
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: logidlestatedisagreement-comments-stale
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: restartsafe-state-reread-branch-unreachable
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: held-page-row-thin-wallclock-margin
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1's six findings are resolved: round 2's review did not re-raise any of them.
Round 2 raised six new findings, F1 to F6, mapped in order to the open keys above.
F1 is the gating one: card 3's `callTimeout` var breaks the injectable-time rule, and its fix adds a parameterized `listOpenIssues`.
F2 and F6 both concern card 3's held-page row and are local corrections.
F3 changes where card 4's `retryLanded` check sits, so its fix needs a fresh review.
F5 asks card 6 to name a reaching row or drop the branch, a choice that changes the card's design.
