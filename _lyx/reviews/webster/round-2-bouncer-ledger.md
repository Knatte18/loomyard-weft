---
round: 2
ledger:
  - key: orch-dry-run-creates-state-lock
    rounds: [1]
    status: resolved
  - key: cleanupreport-applied-comment-misdescribes-field
    rounds: [1]
    status: resolved
  - key: fabricvocab-missing-tmuxkit-testmain
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
---

Round 1 raised two findings, both resolved by round 2: the dry-run state lock no longer creates an absent lock file, and the `CleanupReport.Applied` comment now describes the apply flag.

Round 2 raised `fabricvocab-missing-tmuxkit-testmain` (F1): `internal/fabricvocab` arrived through the merge-in and has tests but no `TestMain` calling `tmuxkit.Main`, which fails the Publish verify.
