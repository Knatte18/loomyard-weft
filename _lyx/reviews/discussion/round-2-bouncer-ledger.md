---
round: 2
ledger:
  - key: d8-board-decision-vs-run-lock
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: in-flight-peer-set-inconsistent
    rounds: [1]
    status: resolved
  - key: d5-bound-overstates-agent-landing
    rounds: [1]
    status: resolved
  - key: d2-forgery-not-compared-with-today
    rounds: [1]
    status: resolved
  - key: d4-post-fallback-verdict-undefined
    rounds: [1]
    status: resolved
  - key: d7-false-arbiter-precedent
    rounds: [1]
    status: resolved
  - key: missing-acceptance-scenarios-d6-d7
    rounds: [1]
    status: resolved
  - key: d8-misses-empty-deploy-cue
    rounds: [1]
    status: resolved
  - key: q4-uncertainty-contradicts-d3
    rounds: [1]
    status: resolved
  - key: d6-prime-form-no-caller-run
    rounds: [1]
    status: resolved
  - key: skip-alignment-local-layer-reopens-gate
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: d5-bound-misses-decision-during-pr-align
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: pr-gate-record-named-decision-record
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d6-bound-narrower-than-decision
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's ten findings: the peer-set definition was confirmed sound and the rest were not re-raised, so they are resolved.
The D8 decision-recording route returns in round 2 as F2: its premise fails before Discussion-Write starts and for every darn run.
Round 2 adds a BLOCKING design finding, `skip_alignment` open to the local layer, which re-opens the agent's auto-landing approval.
It also adds a design gap in D5's bound across `PR-Align`'s wait, and two LOW naming and bound-wording consistency items.
