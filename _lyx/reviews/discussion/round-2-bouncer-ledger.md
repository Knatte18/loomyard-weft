---
round: 2
ledger:
  - key: d12-lyxcwd-breaks-subpath-anchored-hubs
    rounds: [1]
    status: resolved
  - key: d2-marker-misses-hub-building-git
    rounds: [1]
    status: resolved
  - key: d7-buildwithldflags-drops-flags
    rounds: [1]
    status: resolved
  - key: d11-hub-constructors-bypass-tier-purity
    rounds: [1]
    status: resolved
  - key: d11-shared-template-contamination
    rounds: [1]
    status: resolved
  - key: d11-write-definition-incomplete
    rounds: [1]
    status: resolved
  - key: scope-omits-changed-packages
    rounds: [1]
    status: resolved
  - key: d13-allowlist-and-match-shape
    rounds: [1]
    status: resolved
  - key: d7-bound-overstates-agent-deny
    rounds: [1]
    status: resolved
  - key: d6-bound-omits-packages-without-main
    rounds: [1]
    status: resolved
  - key: d12-git-dir-bound-unstated
    rounds: [1]
    status: resolved
  - key: d12-second-local-dotgit-reader
    rounds: [1]
    status: resolved
  - key: d11-copyhub-scan-path-prefix
    rounds: [1]
    status: resolved
  - key: d8-quotes-goal-not-in-goal-section
    rounds: [1]
    status: resolved
  - key: d12-reader-test-import-cycle
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: d12-cwd-resolution-constraint-understated
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d12-walkup-diverges-from-show-toplevel
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-proc-probes-duplicated
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: census-drop-column-unnamed
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: test-budget-list-incomplete
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's fourteen keys are all resolved: round 2 raised nothing on them, and its focus verdicts record D2, D7, D11, D12's subpath anchor and D13 as sound.
Round 2 found six new keys, four of them around D12's move of the `.git` reader into `lyxcwd`.
The BLOCKING one is the import cycle through `gitrepo`'s in-package integration test; the MEDIUM `design` one is the walk-up's divergence from `git rev-parse --show-toplevel`.
The other new keys are the `PATTERN-cwd-resolution` constraint's understatement, D8's duplicated `/proc` probes, the unnamed census column and the incomplete test-budget package list.
