---
round: 1
ledger:
  - key: refusals-without-reaching-test
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: fix-verify-verb-no-behavioural-test
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: await-no-row-for-dead-asking
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: spawned-strand-unrecorded-after-failure
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: missing-model-spec-refusal-no-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: webster-spec-single-fixer-fork
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: loom-plan-spec-fork-only-departure-trailers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: assertions-on-stencil-static-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stale-comment-fork-alone
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
Keys map to the review's findings in order, F1 to F9.
Gating: only F1 (BLOCKING) forces CONTINUE.
The one `design` finding, F4, is LOW.
