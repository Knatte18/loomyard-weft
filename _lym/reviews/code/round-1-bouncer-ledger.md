---
round: 1
ledger:
    - key: beam-member-routing-still-nearest-live
      rounds:
        - 1
      status: resolved
      class: scope
      severity: BLOCKING
      fix: design-wrong
    - key: drift-amend-single-orch-caller-check
      rounds:
        - 1
      status: open
      class: scope
      severity: MEDIUM
      fix: design-wrong
    - key: handback-directive-names-unreached-orch
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: decision-orch-chain-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: batten-parent-name-read-error-fails-tackler-arbiter
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: orch-chain-read-error-refuses-nearest-live-orch
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: parent-name-prompt-wiring-unpinned
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: pair-under-beam-absent-pair-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: overview-doc-states-old-caller-rules
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: hubgeom-doc-go-stale-guard-and-consumer
      rounds:
        - 1
      status: open
      class: design
      severity: BLOCKING
      fix: mechanical
    - key: handback-test-pins-prose-and-tautological
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: handback-missing-stencil-error-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: beam-orch-reap-refusal-text-unpinned
      rounds:
        - 1
      status: open
      class: design
      severity: NIT
      fix: mechanical
---
Round 1 is the first round, and every key is open.

The central gap is that a beam member's notices, asks and escalations still follow the nearest-live walk,
while the diff changes only which callers are admitted.
Related to it are the `lym drift amend` caller check and the hand-back directive's claim about where a stop goes.

The rest are missing or weak tests, two error paths that turn a read failure into a refusal or a lost judgment,
and stale text in `docs/overview.md` and `internal/hubgeom/doc.go`.
