---
round: 1
ledger:
  - key: taskhascommits-wiring-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: taskhascommits-error-path-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: finalize-rejected-token-ordering-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: taskhascommits-reads-every-commit-detail
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 is the first round and has no earlier ledger.
Three findings are test-coverage gaps around the `TaskHasCommits` seam and Finalize's token ordering; each fix is a local test addition.
The fourth finding says `CommitLog` is the wrong call for a boolean, and its fix adds a fabricengine method, so it is an interface change.
