---
round: 1
ledger:
  - key: marked-scope-runs-manual-tier-unlisted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: tier1-marked-case-does-not-check-tier1-settings
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-names-unchanged-cli-go
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: undeclared-tier-refusal-omits-marked-tests
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: test-speed-upkeep-bullet-implies-removed-rule
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round.
The one gating item is that removing the unlisted-tier refusal lets a manual tier's marked tests run without anyone naming the tier.
The rest are a weak test assertion and wording that disagrees with the diff.
