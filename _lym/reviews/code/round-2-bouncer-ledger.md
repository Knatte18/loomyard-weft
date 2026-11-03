---
round: 2
ledger:
  - key: reconcile-unparseable-profiles-untested
    rounds: [1]
    status: resolved
  - key: reconcile-envelope-lacking-untested
    rounds: [1]
    status: resolved
  - key: dry-run-migrated-lacking-unasserted
    rounds: [1]
    status: resolved
  - key: migrateconfig-non-mapping-root-untested
    rounds: [1]
    status: resolved
  - key: walk-lacking-warn-untested
    rounds: [1]
    status: resolved
  - key: differing-former-block-discarded
    rounds: [1]
    status: resolved
---

Round 1 raised six findings: five test gaps (design) and one decision finding on a differing former block.
Round 2 found nothing, so every key is resolved.
The differing former block is now kept and reported in `lacking`, matching the parent's recorded decision.
