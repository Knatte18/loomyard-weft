---
round: 1
ledger:
  - key: removed-guard-bound-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: skill-load-timeout-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card1-doc-ahead-of-code
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first round, with no prior ledger.

- `removed-guard-bound-unstated` (F1): cards 2, 3 and 4 remove or downgrade the unknown-skill guard without stating in Intent what slips through and what bounds it.
- `skill-load-timeout-comment-stale` (F2): the `defaultSkillLoadTimeout` comment still says it bounds one skill's load.
- `card1-doc-ahead-of-code` (F3): card 1 rewrites the `claudeengine` doc before the code it describes changes.

All three are non-gating `consistency` findings.
