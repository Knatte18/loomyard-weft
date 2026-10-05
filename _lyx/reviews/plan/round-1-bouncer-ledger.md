---
round: 1
ledger:
  - key: reset-keep-departs-from-d7
    rounds: [1]
    status: open
  - key: card15-never-marker-bound-unstated
    rounds: [1]
    status: open
  - key: card4-smoke-test-depends-on-operator-settings
    rounds: [1]
    status: open
---

Round 1 raised three findings, none recurring.
`reset-keep-departs-from-d7` (F1, MEDIUM, decision): the plan names `git reset --keep` at the report-head and last-batch-head sites, which D7 does not license.
`card15-never-marker-bound-unstated` (F2, LOW, design): the exemption marker's reach and bound are missing from card 15's Intent.
`card4-smoke-test-depends-on-operator-settings` (F3, LOW, design): card 4's smoke test skips without the operator's deny rule and is not run by verify.
