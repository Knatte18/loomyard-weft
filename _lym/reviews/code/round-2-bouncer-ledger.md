---
round: 2
ledger:
  - key: parent-side-merge-ignores-forced-squash
    rounds: [1]
    status: resolved
  - key: reopen-atonce-flag-untested
    rounds: [1]
    status: resolved
  - key: pr-record-fields-unasserted
    rounds: [1]
    status: resolved
  - key: closed-record-halt-names-noop-pr-review
    rounds: [1]
    status: resolved
  - key: protected-branch-advice-tautological-under-merge
    rounds: [1]
    status: resolved
  - key: prlanding-merge-comment-stale
    rounds: [1]
    status: resolved
  - key: merge-parent-reopen-before-owner-checks
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: ending-mark-writer-doc-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings; the round 2 review raised none of them again, so all six are resolved.
Round 2 raised two new findings.
F1 (`merge-parent-reopen-before-owner-checks`) says a merge parent reopens a closed recorded pull request in `pullRequestLanding` ahead of `ownerChecks`; its fix moves the reopen into `mergeOnGitHub` behind a new `prLanding` flag, so it changes an ordering contract and is marked `design-wrong`.
F2 (`ending-mark-writer-doc-stale`) is two doc comments that still say only PR-Review writes the ending mark.
