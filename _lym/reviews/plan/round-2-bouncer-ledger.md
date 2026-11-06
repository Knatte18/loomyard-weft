---
round: 2
ledger:
  - key: card9-reply-guard-breaks-card7-drop-row
    rounds: [1]
    status: resolved
  - key: card8-socket-text-bypasses-notice-quoting
    rounds: [1]
    status: resolved
  - key: card9-reportengine-doc-unlisted
    rounds: [1]
    status: resolved
  - key: card1-way-forward-overclaims-verbs
    rounds: [1]
    status: resolved
  - key: card5-overview-creel-bullet-verb-list
    rounds: [1]
    status: resolved
  - key: card9-storeinboundlocked-unlisted
    rounds: [1]
    status: resolved
  - key: extends-used-where-card-rewrites-lines
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card1-none-refusal-way-forward-false-for-hubs
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card8-neither-word-answer-and-test-inputs-unnamed
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card5-hubs-registrydir-failure-unstated
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, one BLOCKING; none reappears in round 2, so all are resolved.
Round 2 raised four new findings: an Extends-versus-Edit typing error on cards 2 and 5, a false way forward in card 1's none refusal, unnamed failure mappings and test inputs in card 8, and an unstated registry-directory failure in card 5.
None is gating, so the loop converges.
