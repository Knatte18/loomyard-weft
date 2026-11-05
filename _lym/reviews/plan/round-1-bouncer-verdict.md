---
verdict: CONTINUE
rationale: "Round 1 carries one BLOCKING finding (F1, recipe name declared twice, PATTERN-named-constants) and two gating design findings at MEDIUM (F4, F5); no finding is relabelled."
---

Round 1 is the first round and has no prior ledger, so nothing has converged.

Gating findings:

- F1, BLOCKING, consistency: `boardengine.SketchRecipe` and `shedrun.RecipeRaddle` both declare `"raddle"`. Any BLOCKING finding forces `CONTINUE`.
- F4, MEDIUM, design: no check refuses a part's unknown decision id or PATTERN id before the splice step.
- F5, MEDIUM, design: card 14's Bound overstates what card 24's guard catches.

F3, F8 and the other MEDIUM findings are not design class and do not gate on their own.
F13, F14, F16, F18 and F19 are design class at LOW and do not gate.

No finding's class or severity is relabelled, and the review carries no `## Focus departures` section.
No declared PATTERN change is cited by a finding, so none is gating by that route.
