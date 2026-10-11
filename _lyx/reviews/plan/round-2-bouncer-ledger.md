---
round: 2
ledger:
  - key: sandbox-caller-list-misses-f11-mislabels-f15
    rounds: [1]
    status: resolved
  - key: fabricengine-doc-go-unwire-stale
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: unwire-integration-test-duplicates-engine-coverage
    rounds: [1]
    status: resolved
  - key: floor-value-shapes-pinned-twice
    rounds: [1]
    status: resolved
  - key: migrateconfig-engine-doc-go-not-updated
    rounds: [1]
    status: resolved
  - key: card-5-reconcile-test-asserts-loom-has-no-migration
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised five findings, two of them `design`.
Round 2 raised two LOW findings and no design finding.
Round 2's focus outcomes closed both design keys and the sandbox and `migrateconfig` scope keys.
`fabricengine-doc-go-unwire-stale` survives as F1: the `doc.go` paragraph is covered, but doc comments and file headers on card 6's edited symbols still say unwire always removes.
F2 is new: `TestConfigReconcileInGitRepo` asserts loom has no migration, which card 5 makes false.
