---
round: 1
ledger:
    - key: first-caller-rule-false-premise
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: fixer-unverified-fact-omits-dispute
      rounds:
        - 1
      status: open
      class: consistency
      severity: MEDIUM
      fix: mechanical
    - key: kept-parent-finding-disproved-fact-no-route
      rounds:
        - 1
      status: open
      class: consistency
      severity: MEDIUM
      fix: design-wrong
    - key: carried-finding-no-decision-route
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
---
## Round 1

- `first-caller-rule-false-premise`: the plan-cards stencil, the plan-review rubric and the plan spec tie a new production function to the card of its first production caller.
  The review says no card gate runs the reachability check the rule rests on, and that `pattern/PATTERN-test-only-code.md` allows a later caller.
  The proposed fix removes the rule and corrects the change description.
- `fixer-unverified-fact-omits-dispute`: the fixer step's line on a disproved unverified fact leaves out Dispute.
- `kept-parent-finding-disproved-fact-no-route`: the review format forbids a fix resting on a disproved fact, while parent consolidation requires a kept parent finding to carry the parent's change.
- `carried-finding-no-decision-route`: a carried finding keeps its class, so it cannot be raised as class `decision`.
