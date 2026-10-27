---
round: 1
ledger:
  - key: render-comments-omit-draft-subsection
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: standingcell-comment-omits-draft
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: ready-sentence-ignores-draft-exception
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three comment-wording findings about the new Draft Tasks subsection: F1 (two stale subsection lists in `render.go`), F2 (`standingCell` doc comment) and F3 (the Ready sentence in `doc.go` and `layer.go`).
None is gating, so the loop converges.
