---
round: 2
ledger:
  - key: renumber-insert-kept-range-untested
    rounds: [1]
    status: resolved
  - key: renumber-bare-only-card-write-fault-untested
    rounds: [1]
    status: resolved
  - key: renumber-index-and-heading-lines-untested
    rounds: [1]
    status: resolved
  - key: glyphless-file-failure-paths-untested
    rounds: [1]
    status: resolved
  - key: loom-plan-spec-ambiguous-antecedent
    rounds: [1]
    status: resolved
  - key: card-mention-hyphen-command-rewritten
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: loom-plan-spec-card-id-reported-contradiction
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's five findings are resolved: round 2's review reached each focus site and rejected it as not a defect.
Round 2 raised two LOW findings.
F1 is that `cardMentionRe` matches a hyphen-glued `card N` in a command word and rewrites it.
F2 is a spec sentence that says a card id is never reported although rewritten ids are listed under `rewritten_refs`.
Both fixes are local and the review text gives the exact result.
