```yaml
lens: fix-proposal
outcome: reported
```

fix-proposal lens: I challenged F12's proposal, which was carried out as proposed. The proposal says only the clean-tree reroute clears the fix-here count, and the count is capped "between two send-backs, or before the first".

1. MEDIUM. Changed text: D4, "Only the row's clean-tree reroute clears that count … A re-step, a crash-resume and the resume after the dirty-tree Stuck keep it.", and the criterion "a PR-Review row re-entered after a send-back starts with a fix-here count of zero, and a re-step of the row keeps the count."
   Summary: a backward `lym loom goto` re-runs Code-Review over the whole range, but the count survives, so the first small set afterwards is sent back for no reason. This is the carry-over F12 named, reached by another route.
   Evidence and case: PR-Review halts Stuck on a non-transient failure. Its Stuck is always a Halt, per `contracts/recipes/tapestry-recipe.yaml:522`. The count stands at 3 of `pr_review_small_max` 3. The parent runs `lym loom goto --to Code-Bouncer`, which `internal/shedengine/goto.go:72` (`Goto`) supports. Code-Review re-reviews the range, the run comes down to PR-Review again, and the count is still 3. The parent's next one-finding set then sends the run back although Code-Review has just re-reviewed everything. That costs a full extra generation and spends one of `rerouteCap`'s reroutes. Under the proposal's rule a goto is a re-entry that is not a send-back, so the rule keeps the count. The record names no reset for a goto past PR-Review.

2. LOW. Changed text: D4, "On a clean tree it clears the mark, the fix-here count, the record's `parent_review_asked` flag and its session id. It then returns a Stuck verdict with `Reroute` set."
   Summary: when the reroute cap is reached, the row has already cleared the count before shed halts it. Resuming then gives a fresh `pr_review_small_max` in-place allowance without Code-Review re-reviewing anything.
   Evidence and case: shed decides the cap after the row returns a verdict with `Reroute` set. `internal/shedengine/doc.go:112-116` says "the next reroute halts blocked at the row with a budget-exempt Stuck entry marked reroute_capped". On resume PR-Review re-enters with the mark and the count both zeroed, so the agent can fix up to `pr_review_small_max` more findings in place. This widens the gap the review lists under Rejected as F-D (mark cleared before the cap halt) to the count as well. D4's Bound says the count bounds sets "between two send-backs", and this path breaks that.

Tool calls:
- I grepped `contracts/recipes/tapestry-recipe.yaml` for row edges and grepped `internal/shedengine` for goto. My context had only lines 520-530 of the recipe and none of the shed goto code, and case 1 needed both.
