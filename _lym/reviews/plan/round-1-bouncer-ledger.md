---
round: 1
ledger:
  - key: card2-fixer-report-parser-lenient
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: card1-worktree-tree-ignored-tracked-files
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: card11-arbiter-stalemate-eligibility-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: cards10-12-stale-escalation-cause-enumerations
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: card8-validate-review-texts-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card10-unreadable-ledger-row-unreachable
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card6-first-round-text-misplaced
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-nil-seams-decision-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card10-counts-as-changed-row-equal-ids
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review of the plan, so no earlier ledger exists.
F1 to F9 map in order to the keys above.
F1 and F2 change a contract or an approach (strict parse of the fixer report; seeding the throwaway index from HEAD), so their fixes need a fresh review.
The rest are local text or test-row corrections the findings already spell out.
