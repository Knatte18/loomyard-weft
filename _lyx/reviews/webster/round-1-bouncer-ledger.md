---
round: 1
ledger:
  - key: pairslug-duplicates-weftwarpslug
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: bucketruns-noop-presence-write
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: whole-second-sums-miss-span
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings.
F1 (`pairslug-duplicates-weftwarpslug`) is a duplicated parse that should call `fabricengine.WeftWarpSlug`.
F2 (`bucketruns-noop-presence-write`) is an unexplained no-op write in `bucketRuns`.
F3 (`whole-second-sums-miss-span`) is sub-second boundaries that can leave phases plus sleep a second off the span.
Each fix is fully specified in the finding text.
