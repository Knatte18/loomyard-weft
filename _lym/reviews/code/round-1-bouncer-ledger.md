---
round: 1
ledger:
  - key: retired-row-loom-preflight-missing
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: d6-old-recipe-refusal-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: readme-overview-default-recipe-loom
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: sandbox-suite-seed-recipe-loom
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: battencli-arm-comment-loom
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: recipe-spec-loom-rows-wording
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: tests-keep-loom-recipe-name
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lenses-reviewbudget-docs-over-renamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings, none carried over (first round).
F1 (BLOCKING, design) is the missing `Loom-Preflight` retired-row entry.
F2 to F4 are MEDIUM scope: the untested D6 refusal, and two docs still naming loom as the recipe.
F5 to F8 are LOW consistency leftovers or over-renames.
Every fix is given exactly by its finding text.
