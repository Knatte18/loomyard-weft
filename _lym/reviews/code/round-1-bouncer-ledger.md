---
round: 1
ledger:
  - key: open-project-under-done-project-has-no-readme-section
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: root-order-key-claimed-but-not-applied
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: project-section-priority-and-slug-cells-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all first seen this round.
F1 is the gating one: `projectSections` in `internal/boardengine/render.go` recurses only through non-done projects, so an open project under a done project has no README section.
F2 is a doc over-claim about root-level section order.
F3 is a test gap: no project-section member carries a priority or a body in the golden.
Each fix is a local correction the finding text already specifies.
