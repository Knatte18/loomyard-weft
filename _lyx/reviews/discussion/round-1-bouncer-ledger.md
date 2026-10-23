---
round: 1
ledger:
  - key: batten-restep-reopens-landing-halts
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: arbiterstop-admits-landing-halts
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fixer-failure-exits-unprefixed
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: finalize-fix-cap-size-not-kind
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: overcap-reroute-reaches-no-review-at-zero-budget
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: absent-aligned-tip-skips-finalize-merge
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: overlap-test-on-post-merge-head
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: darn-large-merge-reroute-hides-overlap
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: finalize-escalate-verdict-no-outcome
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: idle-backstop-releases-live-finalize-lock
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: deploy-loses-reviewed-code-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: fixed-verdict-advances-aligned-tip-unchecked
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: lock-bound-omits-retry-pass
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d14-single-table-and-warn-outlet
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reviewed-tip-file-survives-d4
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: skipped-gate-effect-on-failure-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lock-free-merge-in-criterion-unqualified
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: parent-pair-stucks-dropped-from-merge-in
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: open-risks-restate-d10-and-cost-figure
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 19 findings: one BLOCKING, eleven MEDIUM, six LOW and one NIT.
The BLOCKING finding is that batten's per-build re-step undoes the caps the landing halts rely on.
The MEDIUM design findings cluster on three themes: halts that fall outside the landing-stop prefix, bounds on what an unreviewed Finalize fix can let through, and the aligned-tip and overlap computations.
No ledger key existed before this round.
