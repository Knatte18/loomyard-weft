---
round: 1
ledger:
  - key: d9-skills-step-drop-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-removal-set-wider-than-probe
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d1-input-box-matching-rule
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d8-trigger-turn-end-undefined
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d3-no-bound-note-coverage
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: ordering-decision-restated-inconsistently
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d6-voucher-replacement-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d11-stale-stopping-record-unnamed-verb
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: acceptance-criteria-miss-bounds
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: rationale-enumerates-and-repeats
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: docgo-update-covers-only-d9
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eleven findings, none BLOCKING.
The one gating finding is the D9 design gap: the skills step after a compaction is dropped with no Bound line, on a premise the orchengine doc contradicts.
The rest are missing bounds, undefined terms, restated facts and acceptance gaps in `decision-record.md`.
All keys are open; none has been fixed or seen by a second reviewer.
