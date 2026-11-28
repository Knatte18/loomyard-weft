---
round: 1
ledger:
  - key: closed-scope-measure-stop-loops-on-resume
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: new-darnspec-assertions-pin-stencil-prose
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: stop-report-you-run-them-ambiguous
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, all in the darn measurement rule's change.

- `closed-scope-measure-stop-loops-on-resume`: in both make-change fragments under `contracts/stencils/darn/`,
  the closed-scope way forward leads a resumed writer back to the same stop.
  The fix adds a rule for what settles that stop, which is new text for a fresh reviewer.
- `new-darnspec-assertions-pin-stencil-prose`: the added `TestDarnSpec` assertions check static stencil prose.
  Keeping them, or removing them with a `No test seam` record, is the parent's choice.
- `stop-report-you-run-them-ambiguous`: "you" in the stop report's way forward can be read as the report's reader.
