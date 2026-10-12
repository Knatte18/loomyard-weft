---
round: 1
ledger:
  - key: pausecli-pure-test-in-integration-file
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: inject-refusal-text-and-alias-bypass
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: maxinterrupts-edit-glyph-unlisted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: pause-refusal-branches-unreached
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: picker-capture-bound-no-sessioncycler
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: shuttle-interrupt-help-trails-behavior
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised six findings, none recurring.
The BLOCKING one places a pure test in an integration-tagged file; the MEDIUM design one is the Inject interrupt-key refusal bypass through Text and tmux key aliases.
The rest are LOW: an unlisted member glyph, two unreached refusal branches, a wrong bound for engines without `SessionCycler`, and help text landing a commit late.
