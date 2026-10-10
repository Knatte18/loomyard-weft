---
round: 1
ledger:
  - key: d12-lyxcwd-breaks-subpath-anchored-hubs
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d2-marker-misses-hub-building-git
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d7-buildwithldflags-drops-flags
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d11-hub-constructors-bypass-tier-purity
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d11-shared-template-contamination
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d11-write-definition-incomplete
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: scope-omits-changed-packages
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d13-allowlist-and-match-shape
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d7-bound-overstates-agent-deny
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d6-bound-omits-packages-without-main
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d12-git-dir-bound-unstated
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d12-second-local-dotgit-reader
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d11-copyhub-scan-path-prefix
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: d8-quotes-goal-not-in-goal-section
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised fourteen findings and no earlier ledger exists.
The gating ones are the two BLOCKING design findings on D12's `lyxcwd` rule and D2's fixture marker, and four MEDIUM design findings on D7's flag handling, D11's tier-purity bypass, D11's template contamination and D11's write definition.
The remaining MEDIUM findings (scope, D13) are `consistency` and do not gate.
