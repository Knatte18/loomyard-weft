---
round: 3
ledger:
  - key: fanned-fork-rules-drop-spawn-mechanics
    rounds: [1]
    status: resolved
  - key: fan-review-load-refusal-vs-retired-keys
    rounds: [1]
    status: resolved
  - key: never-mid-run-config-bound-false
    rounds: [1]
    status: resolved
  - key: fixer-fan-wiring-undecided
    rounds: [1, 2, 3]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
  - key: bench-measured-scope-misstated
    rounds: [1]
    status: resolved
  - key: d9-named-spawn-omitted
    rounds: [1]
    status: resolved
  - key: d3-model-change-unstated
    rounds: [1]
    status: resolved
  - key: engine-refusal-keys-unnamed
    rounds: [1]
    status: resolved
  - key: fixer-fan-way-forward-ineffective-under-pinning
    rounds: [2]
    status: resolved
  - key: usage-record-authoritative-without-hard-write
    rounds: [2, 3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: plan-lenses-starved-by-fork-call-cap
    rounds: [2]
    status: resolved
  - key: open-risks-omits-boilerplate-removal
    rounds: [2]
    status: resolved
  - key: d4-bench-line-listed-as-change
    rounds: [2]
    status: resolved
  - key: d2-absent-names-refusal-loops
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d2-prespawn-write-vs-live-resume
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-bouncer-refusal-vs-tolerant-reading
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: design-wrong
  - key: fixer-fork-cap-vs-fix-affected-set
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised eight findings and round 2 re-raised only the fixer-fan carrier question; both rounds' other keys are resolved.
Round 2 moved the weight onto D2's per-round usage record, which holds the fan names the fix half and the Bouncer trust.

Round 3 finds the record's design still short on its reader side.
`usage-record-authoritative-without-hard-write` stays open: the write is now stated, but D2 has no rule telling absent names from a recorded empty name, and none for an undecodable record (F2).
Three keys come from the same record.
The refusal on absent names points back at a review row that hands an accepted review on without spawning, so its way forward loops (F1, BLOCKING).
The pre-spawn write and the acceptance-time merge disagree with the live-reviewer resume (F3).
The refusal contradicts the Bouncer's tolerant lens-facts reading and is not scoped to the segments with fan keys (F4).

Two more stay outside the record.
The BurlerFix row has no carrier for the key names its engine refusal must name (F5, continuing `fixer-fan-wiring-undecided`, now LOW).
The six-call fork cap leaves `fix-affected-set` uncovered on the fixer side (F6).

The round-2 keys `fixer-fan-way-forward-ineffective-under-pinning` and `plan-lenses-starved-by-fork-call-cap` resolve: the review's Rejected section holds the former, and its F6 text states Open risks names the Plan-lens cap.
The two mechanical round-2 keys are not raised again and resolve.
