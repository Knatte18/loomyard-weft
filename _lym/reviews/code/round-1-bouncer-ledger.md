---
round: 1
ledger:
  - key: archive-written-outside-anchor
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: bouncer-red-live-and-halt-reap-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: archived-stray-report-unchecked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: red-live-mark-empty-deferred
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: empty-reviewer-fixer-list-accepted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resumed-fix-row-wrong-head
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: copies-expands-vet-guard-runs
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: prose-recipe-lists-omit-hardener
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gate-help-reworded-beyond-copies
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: contained-tests-lack-keep-mark
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: copies-side-behaviours-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: preflight-key-and-segment-settings-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: teardown-spawn-and-proc-kill-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: help-text-rewordings-untraced
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
  - key: pattern-gate-slot-omits-harden-burler-fix
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised fifteen findings, three of them gating design findings at MEDIUM.
The archive path finding changes a struct field and a wiring contract, so its fix is `design-wrong` and needs a fresh review.
The resumed-head finding changes what the fix row records at first entry, so it is `design-wrong` too.
Every other finding names a local correction or a test to add, so it is `mechanical`.
