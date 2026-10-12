---
round: 2
ledger:
  - key: pausecli-pure-test-in-integration-file
    rounds: [1]
    status: resolved
  - key: inject-refusal-text-and-alias-bypass
    rounds: [1]
    status: resolved
  - key: maxinterrupts-edit-glyph-unlisted
    rounds: [1]
    status: resolved
  - key: pause-refusal-branches-unreached
    rounds: [1]
    status: resolved
  - key: picker-capture-bound-no-sessioncycler
    rounds: [1]
    status: resolved
  - key: shuttle-interrupt-help-trails-behavior
    rounds: [1]
    status: resolved
  - key: shuttlepair-readings-test-pins-private-helper
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: pause-settle-serial-per-session
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: told-path-inject-tests-keep-lone-escape
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: card5-pattern-line-count-wrong
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised six findings and round 2 raised four new ones; none recurs.
All six round-1 keys are resolved.
The open items are the BLOCKING test-economy finding on `TestShuttlePairReadings`, the MEDIUM per-session settle in `interruptSession`, a LOW leftover lone-Escape in two told-path Inject tests, and a NIT miscount in card 5.
