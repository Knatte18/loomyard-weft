---
round: 2
ledger:
  - key: holder-glyph-literal-repeated
    rounds: [1]
    status: resolved
  - key: parent-holds-stale-across-owner-reconcile
    rounds: [1]
    status: resolved
  - key: finalize-description-error-text-changed
    rounds: [1]
    status: resolved
  - key: parentalign-moves-scratch-before-row-entry-read
    rounds: [1]
    status: resolved
  - key: optional-seam-names-untested
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: summary-comment-line-figure-stale
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings; round 2 confirmed four resolved: the holder literal, the parent-holds re-read after the owner check, the description error text, and the row-entry read order in Parent-Align.
Two remain, both LOW.
`optional-seam-names-untested` persists because the new subtest takes its cases from the production list, so a dropped name escapes it.
`summary-comment-line-figure-stale` persists because later comment commits moved the figure from 2,574 to 2,576.
