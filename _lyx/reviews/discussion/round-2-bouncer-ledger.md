---
round: 2
ledger:
  - key: operator-ban-contradicts-interactive-discussion
    rounds: [1]
    status: resolved
  - key: restart-never-skips-vs-timeout-skip
    rounds: [1]
    status: resolved
  - key: compact-cycle-unbounded
    rounds: [1]
    status: resolved
  - key: skipped-skill-unbounded
    rounds: [1]
    status: resolved
  - key: directive-role-set-undefined
    rounds: [1]
    status: resolved
  - key: parent-message-recognition
    rounds: [1]
    status: resolved
  - key: rationale-narrative
    rounds: [1]
    status: resolved
  - key: opening-stencil-per-role-undefined
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: request-marker-outlives-watcher
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: skill-skip-bound-disagrees-with-orch-stencil
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d10-exemption-targets-go-text
    rounds: [2]
    status: open
    class: design
    severity: LOW
---

Round 1's seven findings did not reappear in round 2's review, so they are marked resolved.
Round 2 raised four new findings.
F1 (`opening-stencil-per-role-undefined`) is the gating one: D9 does not say which stencil opens each role.
F2 (`request-marker-outlives-watcher`) asks for a recorded request time that survives a watcher restart.
F3 (`skill-skip-bound-disagrees-with-orch-stencil`) concerns the skip bound for `ly:board` and the persistence of a skip.
F4 (`d10-exemption-targets-go-text`) concerns an exemption that names text living in Go.
