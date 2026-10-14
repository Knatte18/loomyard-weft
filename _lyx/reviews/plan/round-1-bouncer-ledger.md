---
round: 1
ledger:
  - key: held-measures-elapsed-on-every-call
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: old-watcher-types-conditioned-notice
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: guard-branches-without-test-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card-12-grep-claim-misses-error-prefixes
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: attach-help-text-stale-on-unix
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: revive-caller-refusal-documented-twice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings (F1 to F6), mapped in order to the keys above.
The gating one is F2: a conditioned revive notice keeps the `.notice` suffix, so a watcher built before the change delivers it without the state check.
F1 (`held` measured unconditionally) changes card 1's contract and card 2's behavior, so its fix needs a fresh review.
F3 to F6 are local corrections the findings already specify.
