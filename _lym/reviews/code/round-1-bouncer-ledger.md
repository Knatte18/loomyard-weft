---
round: 1
ledger:
  - key: remote-tracking-marks-safe-while-git-pushes-elsewhere
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: config-env-attached-form-bypasses-redirect-check
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: relative-push-path-resolved-from-wrong-directory
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: halt-claims-target-repository-unchanged
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: clone-of-remote-git-named-repo-sets-redirect-flag
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failure-exit-without-report-records-no-reason
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: pre-attempt-archive-record-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: lexical-ownership-admits-symlinks-and-failed-clones
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: push-with-u-from-target-passes
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: cancellation-reason-never-asserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: new-guard-branches-have-no-test-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: retired-round-keeps-rerun-record
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: merge-danger-omits-snapshot-format-change
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 mapping of review findings to keys: F1 to `remote-tracking-marks-safe-while-git-pushes-elsewhere`, F2 to `config-env-attached-form-bypasses-redirect-check`, F3 to `relative-push-path-resolved-from-wrong-directory`, F4 to `halt-claims-target-repository-unchanged`, F5 to `clone-of-remote-git-named-repo-sets-redirect-flag`, F6 to `failure-exit-without-report-records-no-reason`, F7 to `pre-attempt-archive-record-untested`, F8 to `lexical-ownership-admits-symlinks-and-failed-clones`, F9 to `push-with-u-from-target-passes`, F10 to `cancellation-reason-never-asserted`, F11 to `new-guard-branches-have-no-test-row`, F12 to `retired-round-keeps-rerun-record`, F13 to `merge-danger-omits-snapshot-format-change`.

First round, so no key recurs.
The push guard in `cluster.go` leaks through remote tracking, `--config-env`, relative paths, symlinks and clones.
The rerun record in `burlerfix.go` misses failure exits without a report and survives a retired round.
