---
round: 2
ledger:
  - key: default-run-selection-undecided
    rounds: [1]
    status: resolved
  - key: repeat-count-vs-fork-replay-skip
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: vanished-session-rule-unbounded
    rounds: [1]
    status: resolved
  - key: read-only-bound-not-by-type
    rounds: [1]
    status: resolved
  - key: no-cutoff-header-claude-term
    rounds: [1]
    status: resolved
  - key: out-undercounts-wording-changes
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: vanished-session-no-drop-wording
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's design findings are settled in round 2: the default run selection is now D10, the vanished-session rule is bounded, and the read-only bound is judged to follow from the adapter types.
The no-cutoff header finding no longer appears.
The fork-replay count remains, narrowed to a per-line wording fix in D2 and its acceptance scenario (F2).
Two new wording findings arrived: Out undercounts the wording changes (F1), and D7's no-drop sentence contradicts its rerun (F3).
All three are consistency class with text fixes the review already gives.
