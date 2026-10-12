---
round: 1
ledger:
  - key: orch-dry-run-creates-state-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: cleanupreport-applied-comment-misdescribes-field
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised two findings; no earlier ledger exists.

- `orch-dry-run-creates-state-lock` (F1): `cleanupHandoffs` acquires the state lock in a dry run, which creates an absent lock file and breaks the `dry-run-shape` decision.
  The `battencli` `takeLock` already meets the rule.
- `cleanupreport-applied-comment-misdescribes-field` (F2): the `CleanupReport.Applied` comment describes the field as "changed anything" while it carries the apply flag.
