---
round: 1
ledger:
  - key: card9-resume-read-error-refusal-unspecced
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: cards3-7-battencli-seam-test-breaks
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card9-loom-start-wording-left-in-cli-and-overview
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card1-result-check-weaker-than-d2-bound
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card6-escalation-test-claims-template-pin
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised five findings (F1 to F5) in the order of the keys above.
F1 is the only gating one: card 9 adds a resume refusal for a recipe or decision read error without a refusal-spec row or a test, and its re-run way forward cannot fix a malformed decision file.
F2 to F4 are scope gaps in cards 3, 7, 9 and 1.
F5 is a wrong claim in card 6 about what its test edit pins.
