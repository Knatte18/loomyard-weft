---
round: 1
ledger:
    - key: tiers-key-path-literal-in-refusals
      rounds:
        - 1
      status: resolved
      class: consistency
      severity: BLOCKING
      fix: mechanical
    - key: timeout-min-wall-clock-term-unpinned
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: planning-other-tiers-unasserted
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: fixer-stencil-no-gate-covers-every-refusal
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: copies-prebuild-monotonic-bound
      rounds:
        - 1
      status: open
      class: scope
      severity: LOW
      fix: mechanical
    - key: gateslot-tier-refusal-text-unasserted
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: failure-block-close-at-total-tests-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: caller-filter-past-command-budget-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: timeout-min-test-keep-marker
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: refusal-lists-miss-doubled-filter
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: interleaved-line-ends-total-tests-summary
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: design-wrong
    - key: filtered-no-test-notice-speaks-for-tier
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: trailing-filter-without-value
      rounds:
        - 1
      status: open
      class: design
      severity: NIT
      fix: mechanical
---
Round 1 is the first round, so no key carries over from an earlier ledger.

The review reports that the diff builds every goal point of the bundled board entry.
Its gating findings are a constraint break in how new refusals name the tiers config key,
two new behaviours that no test pins (the wall-clock term of the tier time bound, and the other tiers planning records),
and a fixer stencil paragraph that applies to more refusals than it should.
The remaining keys are smaller correctness, coverage and doc gaps.

The one `design-wrong` key changes the rule for where an open VSTest summary ends.
