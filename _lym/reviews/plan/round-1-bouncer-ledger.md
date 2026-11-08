---
round: 1
ledger:
  - key: card5-bundles-credential-record-and-login
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d8-recovery-skips-stale-dropped-answer-posts
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
  - key: card5-botscopes-breaks-missingscope-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: nil-livehubs-fallback-unreachable
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card8-holder-branches-without-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card11-missing-setup-refusal-no-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card12-docgo-stale-sentences-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card11-stale-reopen-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card6-lastslackok-wrong-card-credit
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
F1 (card 5 granularity) is the only gating finding; its fix inserts a card and needs a fresh review.
F2 widens D8's recovery to the stale line, the dropped line and inbound answers, which adds behavior.
F3 to F9 are local corrections whose text the findings already fix.
