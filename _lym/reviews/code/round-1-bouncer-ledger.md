---
round: 1
ledger:
  - key: template-default-recipe-comment-lists-names
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1: the `default_recipe` comment in `internal/boardengine/template.yaml` still enumerates recipe names that the `Config.DefaultRecipe` doc dropped.
The fix text fixes the wording, so it is mechanical.
