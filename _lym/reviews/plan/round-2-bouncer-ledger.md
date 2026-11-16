---
round: 2
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
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: stoprecord-attempt-path-uncleared
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: reentry-no-spawn-path-unnamed
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: resume-continuation-unspecified
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: outcome-yaml-schema-unstated
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: weaverrun-wiring-unfilled
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: resume-test-row-duplicate
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised 18 findings, and none of its keys returns in round 2, so all are resolved.
Round 2 raised seven new keys.
The blocker is the begin that holds the state-mutation lease while `saveRunState` acquires it again (F1).
Two MEDIUMs concern the re-entry and resume paths: no named function for the no-spawn path (F3), and an unspecified continuation after a crash (F4).
F2 is the attempt-path gap in `StopRecord` clearing, which round 1's fix did not cover.
The LOWs are the `outcome.yaml` schema (F5), the `WeaverRun` wiring fill (F6) and a duplicate resume test row (F7).
