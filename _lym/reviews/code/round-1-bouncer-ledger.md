---
round: 1
ledger:
    - key: fabricsync-calls-not-gofmt-clean
      rounds:
        - 1
      status: resolved
      class: consistency
      severity: BLOCKING
      fix: mechanical
    - key: wiring-env-read-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: summary-loomcli-timing-claim-contradicts-table
      rounds:
        - 1
      status: open
      class: consistency
      severity: MEDIUM
      fix: mechanical
    - key: webstercli-serial-tests-narrowing-unrecorded
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: mechanical
    - key: loomcli-readonly-tests-still-copy-hub
      rounds:
        - 1
      status: open
      class: scope
      severity: LOW
      fix: mechanical
    - key: seedloomconfig-comment-missing-space
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: verbs-test-header-overstates-skipgit-fixture
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
    - key: ownercheck-spells-pair-shape-inline
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
---
Round 1 is the first round, so every key is new and open.
The gating items are the unformatted `fabricSync` call sites and the missing test of the wiring-time environment read.
The narrowing of the webstercli parallel-tests ask is for the run's parent to rule, not for the fixer to change in code.
