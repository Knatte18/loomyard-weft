---
round: 1
ledger:
  - key: claude-local-md-hides-agents-md
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: runproc-imports-lymcwd-leaf-packages
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: exclude-guard-reads-config-not-seeded-line
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: concurrent-in-place-renders-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: detached-seam-failure-reaches-no-one
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: scope-boarddir-vs-d9-bolt-paths
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: pattern-heddle-text-broken-by-design
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: adopted-resume-reread-unreconciled
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: replayer-set-aside-tag-collides-with-run-slug
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: design-wrong
  - key: owner-read-no-ledger-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-render-removal-blocked-by-exclude-check
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: open-risk-q11-writer-choice
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: geometry-literal-test-vs-constant
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-acceptance-wrong-done
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
The 14 findings (F1 to F14, in the order above) are all open.
Two are BLOCKING design findings: the CLAUDE.local.md render hiding a repository's AGENTS.md from Claude, and runproc importing lymcwd against the declared `PATTERN-leaf-packages` text.
The other gating design findings are the exclude guard reading config rather than the seeded line, unbounded concurrent in-place renders, a detached seam failure reaching no reader, and the added `PATTERN-heddle-instructions` text being broken by D4 and D8.
The remaining findings are local consistency and wording corrections.
