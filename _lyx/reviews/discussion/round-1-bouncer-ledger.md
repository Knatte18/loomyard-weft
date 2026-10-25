---
round: 1
ledger:
  - key: d8-escalation-bypasses-judgment-refusal
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d8-settled-generation-rule-has-no-exit
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-lens-reads-exceed-six-call-cap
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d11-merge-in-fixer-attempt-record-lost
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-landing-approval-is-parents-call
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d8-hard-cap-reescalation-strips-decision-cause
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d8-fix-row-silence-switch-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d10-writer-wait-not-woken-and-silent-stop
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d5-other-fans-and-hub-lens-layers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-accept-carries-answered-findings
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d6-landed-commit-routes-two-ways
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d8-d7-bound-is-prompt-text-only
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 is the first round, so there is no prior ledger to carry forward.
The blocking issue is D8: it escalates ahead of the judgment refusal, so a parent accept can pass a round the refusal would re-judge, and its stated bound is false.
Three MEDIUM design findings follow: D8's settled-generation rule never exits, D6's lens reads exceed the fork's call cap, and D11's carry-across drops the Merge-In fixer attempt record.
F5 asks the parent whether a landing-time approval survives PR-Gate's removal.
The LOW findings are local gaps in D5, D6, D8 and D10.
