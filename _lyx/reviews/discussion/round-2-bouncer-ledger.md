---
round: 2
ledger:
  - key: d5-marker-path-stale-or-unreachable
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d6-resume-partial-or-mixed-round
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d1-d6-d7-join-gate-ordering-single-gate
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: d8-disputed-criterion-presumes-checker
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d9-resolution-at-config-load
    rounds: [1]
    status: resolved
  - key: decisions-without-acceptance-criterion
    rounds: [1]
    status: resolved
  - key: fixer-deadline-vs-waiting-time
    rounds: [1]
    status: resolved
  - key: auto-mode-assumptions-restate-rationale
    rounds: [1]
    status: resolved
  - key: d1-no-shuttle-capability-vs-stop
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: marker-standalone-path-and-cleanup
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: fixer-may-edit-review-file
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d1-first-terminal-event-wording
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 opened eight findings; round 2 did not re-raise four of them (`d9-resolution-at-config-load`, `decisions-without-acceptance-criterion`, `fixer-deadline-vs-waiting-time`, `auto-mode-assumptions-restate-rationale`), so they are resolved.
Four round 1 keys come back in a new form:

- F3 (late marker write) maps to `d5-marker-path-stale-or-unreachable`.
- F4 (resume liveness) maps to `d6-resume-partial-or-mixed-round`.
- F1 (gate cannot fail) maps to `d1-d6-d7-join-gate-ordering-single-gate` and is now BLOCKING.
- F7 (dispute hydration) maps to `d8-disputed-criterion-presumes-checker`.

Four keys are new: F2 (`d1-no-shuttle-capability-vs-stop`), F5 (`marker-standalone-path-and-cleanup`), F6 (`fixer-may-edit-review-file`) and F8 (`d1-first-terminal-event-wording`).
