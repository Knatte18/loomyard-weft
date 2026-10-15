---
round: 1
ledger:
  - key: scalar-or-list-set-refuses-advisor-mappings
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised one finding.
The parent's scalar-or-list set rule refuses list entries that are mappings, and the task's `discussion_advisors` entries are `{name, model}` mappings, so `lyx config loom --set` cannot write the key.
Fixing it changes the set rule's contract, so it needs a fresh review.
