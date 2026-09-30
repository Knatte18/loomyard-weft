---
round: 1
ledger:
  - key: finalize-retry-test-script-contradiction
    rounds: [1]
    status: resolved
  - key: publish-gate-infra-error-and-output-file-unasserted
    rounds: [1]
    status: resolved
  - key: verifyrun-io-discard-copy-pipe-changes-webster-teardown
    rounds: [1]
    status: resolved
  - key: verifyrun-conditional-testmain-file
    rounds: [1]
    status: resolved
  - key: loomengine-card-uses-unconsumed-symbol
    rounds: [1]
    status: resolved
  - key: overview-shared-infrastructure-sentence-missing-verifyrun
    rounds: [1]
    status: resolved
---

Round 1 is the first round, so no earlier ledger exists.
The reviewer raised six findings (F1 MEDIUM, F2–F4 LOW, F5–F6 NIT).
All six were test-specification or precision defects, and the fixer resolved each one in the cards.
I confirmed each fix against the current plan text and found nothing new, so every entry is resolved.
