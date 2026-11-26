---
round: 2
ledger:
  - key: landing-commit-no-durable-source
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: fixer-report-copied-only-at-session-end
    rounds: [1]
    status: resolved
  - key: fixer-copy-names-collide-on-attempt-number
    rounds: [1]
    status: resolved
  - key: atomic-write-temp-files-staged-by-status-commit
    rounds: [1]
    status: resolved
  - key: ask-history-overwritten-after-machine-move
    rounds: [1]
    status: resolved
  - key: never-landed-tags-listed-as-landed
    rounds: [1]
    status: resolved
  - key: extract-write-bound-unstated
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: index-row-written-after-records-push-and-beam-drop
    rounds: [1]
    status: resolved
  - key: fix-review-reports-not-kept
    rounds: [1]
    status: resolved
  - key: pr-findings-rerun-duplicate-copies
    rounds: [1]
    status: resolved
  - key: row-paths-and-extract-paths-differ-in-base
    rounds: [1]
    status: resolved
  - key: landing-commit-message-not-describe-text-alone
    rounds: [1]
    status: resolved
  - key: constraints-misstate-pr-findings-push
    rounds: [1]
    status: resolved
  - key: tag-deletion-exception-describes-no-deletion
    rounds: [1]
    status: resolved
  - key: tag-retention-rule-in-wrong-pattern-entry
    rounds: [1]
    status: resolved
  - key: seeded-ask-record-names-missing-reports
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

Round 1's findings are resolved except two.

- `landing-commit-no-durable-source`: the landing record covers a resumed Finalize,
  but with no record the already-landed path and the archive of a held parent merge still read the parent's head.
- `extract-write-bound-unstated`: the first three bound rules hold;
  the fourth leaves an existing sibling and the rename onto an existing directory unbounded.

One key is new, `seeded-ask-record-names-missing-reports`.
It comes from the round-1 fix for the ask history:
the seed restores the ask record from its tracked copy but not the report files its done asks name.

The review also lists items below its incremental bar; they are not findings and have no key here.
