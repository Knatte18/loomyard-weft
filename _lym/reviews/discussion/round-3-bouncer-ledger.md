---
round: 3
ledger:
  - key: landing-commit-no-durable-source
    rounds: [1, 2, 3]
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
    status: resolved
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
    status: resolved
  - key: landing-record-trusted-after-parent-reset
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: records-side-tracked-gitignore-breaks-fabric-boundary
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
---

- `landing-commit-no-durable-source` (F19) has been open in every round.
  Round 2 found that the paths with no landing record read the parent's head.
  Round 3 finds that the history search replacing that read runs over a task tip Finalize's merge-in has moved,
  and has no rule for squash landings.
- `landing-record-trusted-after-parent-reset` (F20) is new:
  the record is read back after the parent was reset past the merge it names.
- `records-side-tracked-gitignore-breaks-fabric-boundary` (F21) is new.
  It concerns the ignore mechanism in D1, a different defect from `atomic-write-temp-files-staged-by-status-commit`,
  which stays resolved: the temp files are covered, but by a mechanism the PATTERN entry forbids.
- `extract-write-bound-unstated` and `seeded-ask-record-names-missing-reports` are resolved.

The review also lists an item below its incremental bar; it is not a finding and has no key here.
