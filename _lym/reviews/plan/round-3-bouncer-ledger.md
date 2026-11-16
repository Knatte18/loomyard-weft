---
round: 3
ledger:
  - key: reentry-skips-card-gates
    rounds: [1]
    status: resolved
  - key: verify-gate-category-none
    rounds: [1]
    status: resolved
  - key: conflict-record-rel-test-only
    rounds: [1]
    status: resolved
  - key: run-wiring-unlisted
    rounds: [1]
    status: resolved
  - key: stoprecord-never-cleared
    rounds: [1]
    status: resolved
  - key: fixer-record-not-terminal
    rounds: [1]
    status: resolved
  - key: delivered-overwritten-by-save
    rounds: [1]
    status: resolved
  - key: reset-commit-unbounded
    rounds: [1]
    status: resolved
  - key: predicates-missing-resign-arrows
    rounds: [1]
    status: resolved
  - key: per-helper-tests-duplicate-testrun
    rounds: [1]
    status: resolved
  - key: failuretrace-reads-webster-only
    rounds: [1]
    status: resolved
  - key: conflict-path-affected-set-incomplete
    rounds: [1]
    status: resolved
  - key: verify-site-docs-webster-only
    rounds: [1]
    status: resolved
  - key: card23-corrupt-row-unreachable
    rounds: [1]
    status: resolved
  - key: done-commit-failure-untested
    rounds: [1]
    status: resolved
  - key: lymdirs-doc-weaverdirname
    rounds: [1]
    status: resolved
  - key: extends-targets-rewrite-content
    rounds: [1]
    status: resolved
  - key: weaver-slot-name-repeated
    rounds: [1]
    status: resolved
  - key: begin-deadlocks-on-state-lease
    rounds: [2]
    status: resolved
  - key: stoprecord-attempt-path-uncleared
    rounds: [2]
    status: resolved
  - key: reentry-no-spawn-path-unnamed
    rounds: [2]
    status: resolved
  - key: resume-continuation-unspecified
    rounds: [2]
    status: resolved
  - key: outcome-yaml-schema-unstated
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: weaverrun-wiring-unfilled
    rounds: [2]
    status: resolved
  - key: resume-test-row-duplicate
    rounds: [2]
    status: resolved
  - key: retry-allowance-spans-ladder-cycles
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: stale-outcome-survives-restep
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: stopped-batch-gate-calls-unrendered
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stoprecord-writer-unowned
    rounds: [3]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Rounds 1 and 2 are closed: round 3 rejected the round-2 lease-deadlock focus and the StopRecord clearing focus, and its other focus questions surfaced only new gaps.
Round 3 found one design blocker (F1): a re-entry attempt inherits the rung's earlier lost attempts and so gets no same-rung retry.
Its fix adds a `Reentry` marker to `AttemptState` and counts losses from the latest marked attempt.
F2 adds the entry-time removal of a stale `outcome.yaml` and `summary.md`.
F3 adds a `batchGateCalls` target shared by `batchGateSpec` and `checkStoppedBatch`.
F4 is the remaining form of the `outcome.yaml` key-set gap, now as a shared record type and writer.
F5 names the code that sets and saves `StopRecord` before a Stuck return.
