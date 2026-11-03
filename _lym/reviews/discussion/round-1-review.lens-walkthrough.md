```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each Decision on `arras` (and on a nested block for D2) through the code in the manifest.

**Decisions checked**
- D1: a `Cartoon` row with `block`/`block_gate`/`on_done` and no `engine` goes through Parse. Purpose met; `checkRowShape`'s empty-engine refusal has to make an exception for block rows, which D1 implies.
- D2, `arras`: `Cartoon` expands to engine `Block`, skip target Drift-Baseline and `on_done` Discussion-Write. Discussion-Bouncer's empty `on_done` becomes Drift-Baseline and the inlined rows carry path `Cartoon`. Purpose met.
- D2, nested (outer → middle → inner): not met. See finding 2.
- D3, gate holds: step 3a evaluates `always`, then `markEntered(Cartoon)`. The Block producer returns Done, and run.go routes by `OnDone` to Discussion-Write. Purpose met.
- D3, gate does not hold: `enter` appends a skipped entry and routes by the skip target, a change D3 states. Purpose met.
- D3, goto onto `Cartoon`: Goto sets `EnteredRow`, Block returns Done, and the run enters Discussion-Write. Purpose met. Relaunch clears `EnteredRow` and evaluates the gate.
- D4: Discussion-Write returns Stuck with no `on_stuck`, the run halts blocked there, and a resume re-calls it. Purpose met. Discussion-Bouncer's Done routes to Drift-Baseline. The block's end is a Done no history entry records: see finding 1.
- D5: `always` comes from the `BlockGates` table and is set as the block row's EntryGate. Purpose met.
- D6: shed-appended entries get the path from `ProducerDef`, and goto and relaunch entries get it from the expanded routing projection, which D11 carries. Purpose met.
- D7: `mainLine` on expanded `arras` gives Preflight, Cartoon, Discussion-Write, Discussion-Review, Drift-Baseline and so on. `EntryNumbers` over the expanded rows gives the same. On a pass-over, the current row is Drift-Baseline and the step jumps. The budget comes from `env.Segments["Discussion-Review"]`. Purpose met.
- D8: Seed-Child's `prepareRunSettings` with `Recipe=arras` resolves to `tapestry` for `Default` and `Settings`. A profile without an `arras` block is served. Purpose met.
- D9: the `cartoon` rows' config refers only to its own rows and segment (`review_row: Discussion-Burler-Review`, `carry_over: Discussion-Review`, `commit_seam: discussion`). The D2 refusals do not trip. Purpose met.
- D10: in expanded `arras`, Discussion-Bouncer's `on_done` is Drift-Baseline, equal to tapestry's. `checkLoomContract`'s on_done path runs Preflight → Cartoon → Discussion-Write → Discussion-Bouncer → Drift-Baseline → … → Webster, so the build row comes after the record row, and Finalize → Friction-Reflect ends the path at a terminal. Purpose met.
- D11: `loomrecipe.New`, `Routing` and `reviewSegments` parse the raw source today. D11 sends all of them through the expanded recipe. Purpose met as decided.

**Findings**

1. MEDIUM, consistency — D3 against the Goal, D1 and D4: "the block's Done" means two different events.
   - The Goal says the block's Done "is the sub-recipe's end".
   - D1 says the block row's `on_done` is "where the run goes once the block is Done".
   - D4 says "the block is Done when an inlined row whose expanded `on_done` is the block's exit returns Done".
   - D3 records the block row's Done when the run enters the block, before any inlined row runs.
   - Walked through: history and `lym loom status` show `Cartoon` done while Discussion-Write is still running. The block's real end, Discussion-Bouncer's Done, leaves no block-level mark.
   - The board entry says the block "is Done when the sub-run ends done".
   - Evidence: decision-record.md:5, :32, :66, :77; board body "Idea" bullet 1; `internal/shedengine/run.go:462-484` (a Done is recorded and routed by `OnDone`).

2. LOW, design — D1 and D2 against Q9 ("blocks may nest"): a nested block cannot close its sub-recipe, and the order of the two expansion passes is unstated.
   - D1 requires a non-empty `on_done` on every block row, and D2 refuses an inlined `on_done` that names a row outside its own recipe.
   - So a block row cannot be the last row of a sub-recipe: its exit has no row to name and cannot inherit the outer block's exit.
   - D2 does not say whether the inner block is expanded and rewritten before the outer pass's leaving-recipe check. If the check runs after the rewrite, an inner inlined row's rewritten `on_done` names a middle-recipe row. Read against the inner recipe, that is outside its own recipe, and the check would refuse it.
   - Evidence: decision-record.md:32-33, :44, :53; support-log.md:67 (Q9).

Tool calls:
- Read the tapestry Discussion rows' config (lines 136–227) and grepped loomengine for row-index use: the manifest had only the outline of those rows, not their config, which I needed to check D2's refusals against `cartoon`.
