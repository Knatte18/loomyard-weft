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
  - key: callTimeout-package-var-breaks-injectable-time
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: held-page-row-bypasses-pause-gate
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: typeverified-keeps-retrylanded-check
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-session-state-comments-after-conditioned-notice
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
    fix: mechanical
  - key: held-page-row-20ms-margin
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1's six findings do not reappear in round 2, so they are resolved.

Round 2 raised six findings (F1 to F6), mapped in order to the new keys above.
The blocking one is F1: card 3 turns `callTimeout` into a package-level var, which `PATTERN-test-speed` forbids; the review's fix keeps it a const and passes the budget to a new `listOpenIssues` function.
F2 and F6 also sit on card 3's held-page row (pause gate bypassed; thin timing margin).
F3 changes card 4's `typeVerified`/`sendVerified` cut, which needs a fresh look.
F4 and F5 are local corrections.
