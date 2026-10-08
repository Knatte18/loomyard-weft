---
round: 1
ledger:
  - key: generic-rows-caller-coverage-conflict
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: dispatch-skip-bound-unstated-cards-10-11
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: stale-containment-tier-comments-unowned
    rounds: [1]
    status: open
    class: scope
    severity: NIT
---

Round 1 findings, mapped to keys:

- F1 is `generic-rows-caller-coverage-conflict`: card 11's caller coverage flags the receiver of `GenericMethod` as a reference to `Generic`, so the `shapes#Generic` Delete leg (card 1) and arrow leg (card 10) turn blocking, contradicting card 1's claim that no later card changes a shape row.
- F2 is `dispatch-skip-bound-unstated-cards-10-11`: cards 10 and 11 keep their checks out of `ValidateDispatch` without stating what that lets through or what bounds it.
- F3 is `stale-containment-tier-comments-unowned`: two comments in `handle_test.go` still cite both containment tiers, and no card owns them.
