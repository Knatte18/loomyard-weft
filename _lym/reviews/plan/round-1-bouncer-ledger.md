---
round: 1
ledger:
  - key: seed-push-bypasses-destruction-gate
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: hardcoded-branch-names-vs-code-conventions
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: gitrepo-test-rows-spell-side-token
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: interrupted-clone-reconcile-pushes-unseeded-board
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: seeded-board-has-no-upstream
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: abandon-seeded-clone-no-reaching-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: never-pushed-row-passes-without-namespace
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card9-misses-namespace-sites
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: deadcode-guard-producer-before-consumer
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: join-fetch-skips-remote-refs-lock
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: join-reset-reads-origin-head-for-records
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: transient-refusal-way-forward-unreached
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card4-edit-targets-unchanged
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card1-tests-name-no-tier-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: seed-bullet-unreachable-reason
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 15 findings against the plan; none is resolved yet.
The BLOCKING three are the seed push bypassing the destruction gate, the branch-name literals against `PATTERN-code-conventions`, and the side token in card 3's gitrepo test rows.
The design-class MEDIUMs concern the unseeded-board reconcile path, the missing upstream on the seeded board, test reach for abandon and never-pushed rows, card 9's namespace coverage, and card batching against the deadcode guard.
