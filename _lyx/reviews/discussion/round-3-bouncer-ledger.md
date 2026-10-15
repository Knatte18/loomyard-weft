---
round: 3
ledger:
  - key: d2-pattern-entry-overview-headroom
    rounds: [1, 2]
    status: resolved
  - key: d4-route-does-not-carry-a-drop
    rounds: [1]
    status: resolved
  - key: d6-plan-review-acceptance-unbounded
    rounds: [1]
    status: resolved
  - key: edit-wider-than-board-lock
    rounds: [1, 2]
    status: resolved
  - key: d5-darn-no-route-for-own-entry-step
    rounds: [1, 2]
    status: resolved
  - key: d4-darn-omission-self-contradiction
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: scope-and-run-settings-unpinned
    rounds: [3]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: d1-claude-md-rule-without-rationale
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 3 raised no design finding.
The PATTERN headroom, the board-lock wording and the darn-path routing keys from rounds 1 and 2 are resolved.
The three open items are local text corrections: D4's contradictory ordering for the darn writer, the missing definition of scope and the run settings exclusion, and the missing reason in D1's CLAUDE.md rule.
