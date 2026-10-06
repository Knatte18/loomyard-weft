---
round: 3
ledger:
  - key: d9-skills-step-drop-unbounded
    rounds: [1, 2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: d4-removal-set-wider-than-probe
    rounds: [1]
    status: resolved
  - key: d1-input-box-matching-rule
    rounds: [1, 2, 3]
    status: open
    class: design
    severity: MEDIUM
  - key: d8-trigger-turn-end-undefined
    rounds: [1, 2, 3]
    status: open
    class: scope
    severity: LOW
  - key: d3-no-bound-note-coverage
    rounds: [1]
    status: resolved
  - key: ordering-decision-restated-inconsistently
    rounds: [1, 2]
    status: resolved
  - key: d6-voucher-replacement-unbounded
    rounds: [1, 2]
    status: resolved
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
    rounds: [2, 3]
    status: open
    class: decision
    severity: BLOCKING
  - key: d5-removals-not-logged
    rounds: [2]
    status: resolved
  - key: constraints-cite-d14-patterns-it-omits
    rounds: [2]
    status: resolved
  - key: d8-rationale-contradicts-d9
    rounds: [2]
    status: resolved
---

Round 3 has five findings.
The prune-ordering, D6, D5 logging, D14 pattern-citation and D8-rationale keys did not recur and are marked resolved.

Still open:

- D12 (F1, BLOCKING): the record states the gate-self-check-parity verb is not delivered; it must either name the verb or amend the pattern.
  Open since round 2.
- D1 (F2, MEDIUM design, F3 LOW folded in): the contains-match is unbounded for a short sent text or a shared prefix, and the needle length and settle are not pinned.
  Open since round 1, sharpened each round; this is the gating design key.
- D8 (F4): the no-turn-end case and the turn end being read are undefined.
  Open since round 1.
- D9 (F5): D9 and D10 disagree on where a compaction reload starts, and the `doc.go` update has no acceptance line.
  Open since round 1, now a wording and coverage gap rather than an unbounded drop.
