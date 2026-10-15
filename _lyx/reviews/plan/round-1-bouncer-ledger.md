---
round: 1
ledger:
  - key: card1-breaks-webster-recover-fixtures-until-card7
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: run-merriambase-refusal-triggers-unreachable-behind-entry-gate
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: advisor-way-forward-sets-list-key-to-scalar
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: dropped-generic-way-forward-strips-orch-refusal
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: requires-rows-not-reached-through-fake-engine-set
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised five findings against the plan, all open.
F1 (BLOCKING, design) is a card-order defect: card 1 makes webster's specs carry the resolved engine, but the fixtures that keep the recover tests green move only in card 7.
F2 concerns card 10's refusal triggers that card 9's entry gate makes unreachable.
F3 and F4 concern card 11's way-forward mechanics, a list-valued `discussion_advisors` key and the orch losing the generic clause, and both need new text.
F5 concerns card 11's tests not reaching the advisor and MultiLLM Requires rows through a fake engine set.
