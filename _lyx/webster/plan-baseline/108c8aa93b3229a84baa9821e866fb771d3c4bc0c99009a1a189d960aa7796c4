# Card 14 — gate-quiescence

**Edit:**
- `internal/loomrecipe/gatequiescence_test.go`
- `contracts/recipes/loom-recipe.yaml`

**Intent:**
Decisions D7 and D9: the declined gate-quiescence decision is re-opened on the ground `shuttle-state` built, and its guard is rewritten to match.
A gated row may authorize fork subagents only on a BurlerRound row, where the forking half is the reviewer and its only gate is the review-parse entry;
writer rows (DiscussionWrite, PlanWrite, Describe, PRRework) still never fork, and the fixer never forks, which `TestEngine_Run_ForkSubagentsSpecWiring` already pins.
`gatequiescence_test.go` is rewritten around a rule function over a parsed recipe and the wiring's env-supplied fans (a row-name to fan map, as `Env.RowClusterFans` carries them): it flags a gated row that receives a fan, from its profile's `cluster-fan` or from the map, and whose engine is not BurlerRound, and a gated row whose engine is neither a writer engine nor BurlerRound.
`TestNoGatedRowAuthorizesForkSubagents` runs it over the shipped recipe with fans set for the Discussion-Review and Plan-Review rows through the `loomshed` row names, and over hand-authored recipes: a gated writer row receiving a fan fails and a gated BurlerRound row with a fan passes.
It also pins that no shipped row carries a literal `cluster-fan`, Webster-Burler included (D1, D9).
The shared failure message names the new ground: a turn end whose Stop payload reports an outstanding fork is a wait, so no gate runs while a fork is live, and the burler fork audit's exact count is the backstop.
`TestWebsterRoundCarriesNoGateKey`'s doc comment drops its "could legitimately grow a fan" premise.

The Webster-Burler comment ending "Single reviewer, matching both shipped segments" in `loom-recipe.yaml` states instead that Webster-Review stays solo because forks run no git and its subject exists only through git, while Discussion-Review and Plan-Review take their fan from `loom.yaml`.

No shuttle test changes: `TestPollEventsTick_ShellExpiry`'s row "a fork beside payload-reported shells keeps the gated run waiting" already pins D7's shuttle wait rule, a gated run with its output file present and a passing gate still waiting at the shell bound while a fork is outstanding.

**ImpactSummary:** Test and recipe-comment changes only; the shipped recipe's rows, engines and gates are unchanged.
