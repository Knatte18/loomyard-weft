---
round: 1
ledger:
  - key: codex-strand-no-delivery
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: orchengine-reminder-docs-missed
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: interrupt-target-record-failure-after-key
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: noturn-interruptguard-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: llm-smoke-waiter-lifetime
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unreadable-pause-refusal-unwrapped
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: extends-on-rewritten-refs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hookread-resign-arrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: hookreadexit-test-duplicates-cli
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-hooks-refusal-typed-notice-claim
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: hookwait-refusal-external-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: capabilities-header-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: prepare-test-header-stale
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 13 findings, none carried from earlier rounds.
The BLOCKING one is a decision gap: how a strand with no provider tag, a hand-added Codex session included, is treated by card 8's `PaneMail`.
Two design MEDIUMs concern test reach (card 7) and an LLM scenario resting on an undocumented waiter lifetime (card 18).
The rest are local doc, glyph, Extends and refusal-text corrections.
