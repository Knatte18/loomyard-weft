---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md` D4, check that every engine-consulting `Runner` method is assigned a way to pick its engine, including `Runner.InterruptSettle`, which takes no argument and is called per session by `tenderengine`'s `settle` over a pause that may span runs of two engines. Settle which engine each caller reaches."
  - "In D7, check what bounds `websterengine`'s use of the default engine for fork audits and for `TurnEndedAfter` parsing, given that `Start` and `Wait` resolve the spec's and the recorded engine. Settle whether the record states when the two can diverge and who must act then."
  - "In D4's Attach bullet and D8, check what an attach does when the recorded engine does not meet the spec's `Requires`, including the `forks` implied by `ForkSubagents`. Settle whether D8's bound that `Requires` never lets a run skip a check holds on the attach path."
---

Each entry names the Decision to read and the question to settle.
