---
round: 1
ledger:
  - key: landing-commit-no-durable-source
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: fixer-report-copied-only-at-session-end
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: fixer-copy-names-collide-on-attempt-number
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: atomic-write-temp-files-staged-by-status-commit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ask-history-overwritten-after-machine-move
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: never-landed-tags-listed-as-landed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: extract-write-bound-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: index-row-written-after-records-push-and-beam-drop
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fix-review-reports-not-kept
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pr-findings-rerun-duplicate-copies
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: row-paths-and-extract-paths-differ-in-base
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: landing-commit-message-not-describe-text-alone
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: constraints-misstate-pr-findings-push
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tag-deletion-exception-describes-no-deletion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tag-retention-rule-in-wrong-pattern-entry
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

First round, so every key is new and open.

The weight is on where the kept records come from and when they are copied:
the landing commit's source, the point and naming of fixer and fix review report copies, the ordering of the index row,
and what survives a concurrent commit or a machine move.
The read side has two further design gaps: how a tag of a run that never landed is listed, and what bounds `--extract`'s writes.
The consistency findings are local wording and path corrections.
