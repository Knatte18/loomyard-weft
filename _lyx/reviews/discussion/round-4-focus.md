---
round: 4
focus:
  - "In `_lyx/discussion/decision-record.md` D8, settle whether a recorded `continue` can deliver a refused judgment's BLOCKING findings to the next round's fixer. The perch order in `contracts/recipes/loom-recipe.yaml` (Discussion rows) runs Bouncer, Burler-Review, Burler-Fix, Bouncer, so the fixer of round n runs before round n's judge. A CONVERGED judge writes an empty `focus` list (`internal/shedadapters/bouncerprompt.go`, `bouncer-template-judge.md`), and the circling decision file holds only round, decision, cause, settled and automatic (`internal/shedadapters/circling.go`). Name the row or file that carries the findings, or check that D8 no longer claims a carrier, and that the support log's Rejected alternatives and D8 do not contradict each other."
  - "In D8, settle what happens when the judgment refusal still holds at or past the hard cap. Round numbers carry on past the segment budget, so no re-judge can be spent there, and D8 offers accept only at the hard cap while a `continue` re-escalates with cause `decision`. Check that the record states a terminal state for that case, that the state names the way forward, and that an acceptance criterion pins it."
  - "Check that any new text D8 gains for the two questions above keeps the attack-surface bound stated: what a recorded accept or `continue` can now let through, and what bounds it (the hard cap, one escalation per judged round, the refusal's checks)."
---

The round-3 review settled the accept's two BLOCKING checks, the dropped-key check on a `continue`, the Added-after-Discussion snapshot file and the D10 no-parent stop; a fixer's edit to D8 may disturb them, so the reviewer reads D8 whole.
