---
round: 1
ledger:
  - key: protected-completion-not-bounded-to-prime
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: records-settled-zero-value-two-repo
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: own-unsettled-merge-in-not-own-leftover
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: merge-state-test-mirror-missing
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: records-commits-outside-three-primitives
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: takeover-push-bypasses-prime-guard
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: marker-change-refusal-after-back-merge
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card10-conflicted-index-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: seam-docs-skip-wording
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: protected-push-test-duplicates-push-error-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: records-repo-root-asserted-twice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resolve-within-rows-repin-cwd-gate
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-two-repo-promise-vs-card11
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: prime-branch-key-no-stated-bound
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: sleep-not-a-new-field
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 opened fifteen findings, all open.
Two are BLOCKING: the protected completion reaches every push instead of the prime's records push alone (F1),
and the `RecordsSettled` zero value marks every two-repository parked merge unsettled (F2).
F7 is the one gating design finding at MEDIUM.
The remaining findings are local scope, test-economy and wording corrections whose fix text is already fixed by the review.
