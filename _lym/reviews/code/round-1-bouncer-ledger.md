---
round: 1
ledger:
  - key: dismiss-picker-escape-outside-pane-text-injection
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: dismiss-picker-running-turn-guard-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: no-box-reprompts-reset-semantics-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: failed-dismissal-key-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: conditioned-send-no-input-box-unread
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: autocompact-fixture-invented-shape
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, none carried from an earlier round.
The one gating finding is the Escape dismissal in `dismissPicker`, which adds a key-delivery path `PATTERN-pane-text-injection` does not allow, with no PATTERN edit and no `Pattern-Departure:` trailer.
The remaining five are test-coverage gaps (running-turn guard, reset semantics), a failed-key hold that escapes the no-input-box bound, an unread `NoInputBox` on conditioned sends, and a hand-drawn fixture glyph.
