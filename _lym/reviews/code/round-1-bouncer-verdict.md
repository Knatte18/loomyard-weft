---
verdict: CONTINUE
rationale: "Round 1 carries two gating design findings at MEDIUM, F1 (the card fixer's SetNotifier never fires) and F2 (no test reaches newShed's notify wiring); no relabels."
---

Round 1 has 8 findings, none BLOCKING.
F1 and F2 are class `design` at MEDIUM, so the decision rule gives `CONTINUE`.
F3 to F8 are LOW or NIT and not gating; F4 is `design` at LOW and F7 is a `decision` finding deferred to the parent.
No finding was relabeled, and the review has no focus departures needing more than a note:
the single departure, on focus item 3, is ratified because the notifier's non-firing (F1) is the prior question and makes the routing question moot.
