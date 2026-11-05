---
round: 2
ledger:
  - key: corrupt-template-base-wedges-reconcile
    rounds: [1]
    status: resolved
  - key: template-base-commit-paths-untested
    rounds: [1]
    status: resolved
  - key: reconcile-envelope-keys-unasserted
    rounds: [1]
    status: resolved
  - key: orch-mail-lines-unpinned
    rounds: [1]
    status: resolved
  - key: followtemplate-sequence-rows-missing
    rounds: [1]
    status: resolved
  - key: base-read-failure-and-restore-untested
    rounds: [1]
    status: resolved
  - key: reencode-folds-long-flow-mappings
    rounds: [1]
    status: resolved
  - key: test-split-from-doc-comment
    rounds: [1]
    status: resolved
  - key: docs-still-say-never-fills
    rounds: [1]
    status: resolved
  - key: corrupt-base-parsed-when-ignored
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 2's review settled every round 1 key through its focus verdicts, so all nine are resolved.
The one new finding, F1, maps to `corrupt-base-parsed-when-ignored`: `FollowTemplate` parses the base when `haveBase` is false, so a corrupt base fails a reconcile that would not read it.
Its fix is local and fully stated by the finding, so it is mechanical.
