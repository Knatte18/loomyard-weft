---
round: 2
ledger:
  - key: d9-skills-step-drop-unbounded
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d4-removal-set-wider-than-probe
    rounds: [1]
    status: resolved
  - key: d1-input-box-matching-rule
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-trigger-turn-end-undefined
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
  - key: d3-no-bound-note-coverage
    rounds: [1]
    status: resolved
  - key: ordering-decision-restated-inconsistently
    rounds: [1, 2]
    status: open
    class: consistency
    severity: NIT
  - key: d6-voucher-replacement-unbounded
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d11-stale-stopping-record-unnamed-verb
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-miss-bounds
    rounds: [1]
    status: resolved
  - key: rationale-enumerates-and-repeats
    rounds: [1]
    status: resolved
  - key: docgo-update-covers-only-d9
    rounds: [1]
    status: resolved
  - key: d12-gate-parity-claim-without-verb
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
  - key: d5-removals-not-logged
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: constraints-cite-d14-patterns-it-omits
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d8-rationale-contradicts-d9
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
---

Round 1's D4, D3, D11, acceptance-criteria, rationale and `doc.go` findings did not recur in round 2 and are marked resolved.
D4 was checked in round 2 and found not a defect.

Still open from round 1:

- D1 matching rule: round 2 sharpens it into two design gaps, the untested leading-part length (F1) and the unstated read timing after Enter (F2).
  This is the gating key.
- D9: now a conflict with R5 and the `doc.go` statement (F3).
- D8: two undefined boundary cases (F4).
- D6: the criterion is narrower than the bound (F8).
- Prune ordering stated three times (F9).

New in round 2: D12's pattern claim without a verb (F5), D5 logging (F6), D14 pattern citations (F7), and D8's rationale against D9 (F10).
