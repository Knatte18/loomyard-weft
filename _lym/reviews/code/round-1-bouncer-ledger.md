---
round: 1
ledger:
  - key: recordwrite-envelope-reason-way-forward
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: errorenvelopeline-matches-fenced-quote
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: isbareslug-rejecting-side-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: board-skill-bare-slug-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: board-help-omits-envelope-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none seen before.
The two gating ones are the Record-Write Stuck reason, whose way forward strands the slug and is false for a pawl beam project,
and the envelope detector, which also refuses a body that quotes an envelope inside a fenced block.
The other three are a missing rejecting-side test for `isBareSlug` and two stale docs: the board skill and the merge and upsert help.
