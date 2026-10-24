---
round: 2
ledger:
  - key: untuned-darn-defaults-migrate-to-loom
    rounds: [1]
    status: resolved
  - key: commit-discussion-snapshot-order-untested
    rounds: [1]
    status: resolved
  - key: new-key-moves-unpinned
    rounds: [1]
    status: resolved
  - key: configcli-doc-retired-key-example
    rounds: [1]
    status: resolved
  - key: reroute-reviewed-code-budget-zero-untested
    rounds: [1]
    status: resolved
  - key: snapshot-refusals-lack-way-forward
    rounds: [1]
    status: resolved
  - key: redundant-perch-hard-cap-row
    rounds: [1]
    status: resolved
  - key: former-default-drop-unreported-prime-seeded
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings; round 2's review confirms the gating one (untuned darn defaults migrating into loom) is fixed, and none of the other six reappear.
Round 2 raised one LOW finding: the drop line for a former default in a prime-seeded source lands on a target file that `moveKeys` does not list as changed, so the report is lost, though the value is dropped correctly.
