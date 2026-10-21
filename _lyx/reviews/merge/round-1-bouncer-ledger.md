---
round: 1
ledger:
  - key: gates-open-map-single-set-drops-other-lists
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: darn-fixture-retired-verify-substring-accident
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised two findings from the merge of the parent's sparse hub-base writes with the task's open-map `gates` declaration.

- `gates-open-map-single-set-drops-other-lists` (F1): a hub `--set gates.<name>` writes a sparse base holding one gate, and the open map replaces the template's whole, so other gates lose their lists.
  The fix changes how `gates` is declared (per-name map), so it needs a fresh review.
- `darn-fixture-retired-verify-substring-accident` (F2): the parent's darn fixture plants `verify: make verify`, which after the task retired darn `verify` matches only inside `tree_scope_verify`; no test sends a retired key through the walk strip.
  The fix is a local test correction the finding text fixes.
