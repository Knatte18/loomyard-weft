---
round: 1
ledger:
  - key: fresh-subagent-fork-literal-not-constant
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: meta-without-kind-reads-fresh
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: fix-round-named-fresh-delegate-fails-as-named-fork
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fresh-subagent-test-bypasses-public-audit-surface
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: isfork-term-untested-alone
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: spawnboundary-tests-need-keep-markers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: forkauditor-docs-describe-forks-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: audit-header-and-docs-say-fork-transcripts
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first ledger, so nothing carries forward.
The review's eight findings map one to one onto the keys above, F1 to F8 in order.
F1 is a repeated `"fork"` literal beside `forkSubagentType`, BLOCKING under `PATTERN-code-conventions`.
F2 and F3 are behavior gaps in the fix-round audit: a meta with no recorded kind fails open, and a named fresh delegate still counts as a named fork.
F3 adds a provider-neutral `ForkAudit` field, so its fix changes an interface and is `design-wrong`.
F4 and F5 are test placement and a missing row; F6 to F8 are keep markers and stale docs.
