---
round: 1
ledger:
    - key: card-scope-drops-dependents-tier1-guards
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: failed-dependents-listing-narrows-silently
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: implementer-stencil-guard-deviation-misses-tier-run
      rounds:
        - 1
      status: open
      class: consistency
      severity: MEDIUM
      fix: mechanical
    - key: gatecli-doc-vet-step-says-card-packages
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: map-card-widened-scope-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: card-dependents-success-row-redundant
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
    - key: dependents-unclean-seed-listed-twice
      rounds:
        - 1
      status: open
      class: design
      severity: NIT
      fix: mechanical
---
Round 1 reviewed the change that widens the `--card` gate scope to the packages that depend on the card's packages.
The main defect is in how that widening meets the guard step:
`scanGuards` treats every package of the widened scope as covered, whatever tiers the call lists,
so guard tests of a higher tier in a dependent package run nowhere at the card gate.

The other findings follow from the same widening:
a failed dependents listing is only logged,
the implementer stencil and the `# Vet step` doc section still describe the narrower scope,
`--map --card` has no test over a widened scope,
one test row repeats existing coverage,
and `dependentsFor` compares cleaned and raw spellings of a seed.

The reviewer left two cases undriven: non-Go card targets, and dependent packages whose files are all behind a build tag.
