---
round: 2
ledger:
  - key: d9-budget-exit-misnamed
    rounds: [1]
    status: resolved
  - key: d8-at-entry-settle-unbounded
    rounds: [1]
    status: resolved
  - key: d7-at-entry-sentence-misdescribes-bound
    rounds: [1]
    status: resolved
  - key: d2-seed-precedence-no-criterion
    rounds: [1]
    status: resolved
  - key: d9-rationale-carries-rejected-alternative
    rounds: [1]
    status: resolved
  - key: d9-refusal-at-spent-budget-no-rejudge
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: d7-no-unchecked-path-to-done-overstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d10-crash-replay-judgment-no-usage-entry
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's five findings no longer appear in round 2's review, so they are resolved.
The D9 budget exit now agrees across its three sites.

Round 2 raised three new findings.
The gating one is that a refusal at a spent budget blocks the run at once, with no re-judge and no parent escalation.
The other two are D7's "no unchecked path to Done" sentence, which the escalation accept contradicts, and D10's usage entry, which a judgment settled from disk after a crash never gets.
