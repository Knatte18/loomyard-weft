---
round: 2
ledger:
  - key: claude-local-md-hides-agents-md
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runproc-imports-lymcwd-leaf-packages
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: exclude-guard-reads-config-not-seeded-line
    rounds: [1]
    status: resolved
  - key: concurrent-in-place-renders-unbounded
    rounds: [1]
    status: resolved
  - key: detached-seam-failure-reaches-no-one
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: scope-boarddir-vs-d9-bolt-paths
    rounds: [1]
    status: resolved
  - key: pattern-heddle-text-broken-by-design
    rounds: [1]
    status: resolved
  - key: adopted-resume-reread-unreconciled
    rounds: [1]
    status: resolved
  - key: replayer-set-aside-tag-collides-with-run-slug
    rounds: [1]
    status: resolved
  - key: owner-read-no-ledger-unstated
    rounds: [1]
    status: resolved
  - key: stale-render-removal-blocked-by-exclude-check
    rounds: [1]
    status: resolved
  - key: open-risk-q11-writer-choice
    rounds: [1]
    status: resolved
  - key: geometry-literal-test-vs-constant
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-acceptance-wrong-done
    rounds: [1]
    status: resolved
  - key: in-place-render-reread-at-compaction
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: heddle-json-read-no-bolt-lock
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: render-verb-target-unbounded
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised fourteen findings, two BLOCKING.
Round 2 reopens four of them at lower weight: the `.claude/AGENTS.md` variant of the hidden-AGENTS.md concern, the leaf-packages background edit, the geometry-literal test exception, and the wrong detached example in D6.
The other round-1 keys are resolved.
Round 2 adds three keys: a Claude re-read of the in-place render at compaction, an unlocked read of `heddle.json` during Bolt checkout moves, and an unbounded render target.
The two MEDIUM design keys are first raised in round 2, so Go carries them into Open risks.
