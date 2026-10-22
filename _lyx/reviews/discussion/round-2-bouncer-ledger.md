---
round: 2
ledger:
  - key: d5-warn-breaks-lenses-leaf-imports
    rounds: [1]
    status: resolved
  - key: d4-base-scaffold-pins-base-prose
    rounds: [1]
    status: resolved
  - key: worktree-template-equal-drop-in-hub-nonexistent
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d3-way-forward-leaves-worktree-lens-set
    rounds: [1]
    status: resolved
  - key: local-layer-lens-set-guard-message
    rounds: [1]
    status: resolved
  - key: fan-superset-criterion-weaker-than-d2
    rounds: [1]
    status: resolved
  - key: hub-worktree-set-lens-set-contradictory-way-forward
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's three design keys (D5 leaf imports, D4 scaffold, D3 way forward) were not re-raised in round 2; the review's focus verdicts found each fix sound.
The local-layer and fan-superset consistency keys were likewise not re-raised.
Round 2's F1 reopens the template-equal-drop key: the reconcile acceptance criterion still opens with "Under a hub file" and names a stack that cannot exist; the review supplies the replacement bullets.
F2 is new: a hub-worktree `lyx config lenses --set lens_set=<name>` is rolled back with a generic way forward no value satisfies; the review supplies the added sentence and criterion.
