```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked pairs of decisions that act on the same state for contradictions.

**Pairs checked**
- D1 × D2 (a block row's `on_done` and engine `Block` stamped after Parse): held. Nested blocks break; see F3.
- D2 rewrite of empty `on_done` × D2 refusal of an inlined edge leaving its recipe, with nested blocks: broken (F1).
- D2 × D3 (skip target set by expansion, read at run time): held.
- D2 segment refusal × D7 review budgets: held.
- D3 block row "records Done" at entry × D4 "the block is Done when an inlined row … returns Done" × Goal: broken (F2).
- D3 pass-over × D3 goto onto an inlined row (`EnteredRow`, `gotoAdmitted`): broken (F4).
- D3 goto onto the block row × D5 gate: held. A goto bypasses the gate, and Relaunch clears `EnteredRow`, so the gate is evaluated again (goto.go:124, 244).
- D3 skip × `pause_after` (`routedState` keys on `def.Name`): held. A skip pauses at the skip target, an entry at the sub entry.
- D3 × D6 (skipped and goto entries carry the block path): held through D11's projection.
- D3 × D7 (pass-over moves from the block's step straight to the exit's step): held, because `mainLine` follows the expanded `on_done` (progress.go:33-60).
- D7 × D2 file order × `EntryNumbers`: held. The numbering is Preflight, Cartoon, Discussion-Write, Discussion-Review, matching the acceptance criterion.
- D4 × D2 (inlined `on_stuck` empty means a halt): held.
- D9 × D2 refusals (names, segments, top-level keys, `terminals` unread): held for `cartoon` inside `arras`.
- D9 pin × D10 pin × peer `approve_seam: discussion`: held. The expanded Bouncer's `on_done` is rewritten to Drift-Baseline, which equals tapestry's.
- D8 × D2 (`profile_recipe` refused in a sub-recipe) × D10 (`arras` sets it): held.
- D11 × D7 (the review-segment budget must come from the expanded rows; `Routing` today parses the raw source in `reviewSegments`, loomrecipe.go:127): held as D11 states it.
- D11 × D9 (`cartoon` resolvable by name but not seedable): held. `New("cartoon")` is refused because the recipe declares no `contract: loom` (loomrecipe.go:61).
- Constraint `PATTERN-shed-recipe-registry` × D5: held.

**Findings**

**F1 — MEDIUM, design. D2: with nested blocks, the order of rewrite and refusal is undefined.**
- D2 rewrites "every empty `on_done` to the block's exit" for the inlined rows, and refuses "an inlined row whose non-empty `on_done`, `on_stuck` or `on_reroute` names a row outside its own recipe". It never says whether the inner block is expanded first.
- Take a middle recipe M that holds block I, inside outer block O:
  - If the outer rewrite runs over rows that are not yet expanded, I's last row is rewritten to O's exit. M's rows after I are then skipped.
  - If the inner expansion runs first, I's last row already carries `on_done` = I's exit, which is a row of M. I's block row carries `on_done` = I's entry. The outer refusal then sees inlined edges naming rows outside "its own recipe" and refuses D2's own rewrite.
- Neither D2 nor D4's Done condition ("expanded `on_done` is the block's exit") says which block's exit applies.
- Evidence: decision-record.md:44, 49-54, 77; support-log.md:67 (Q9 allows nesting).

**F2 — MEDIUM, consistency. "Done" means two different events.**
- The Goal and the board say the block is "one producer whose Done is the sub-recipe's end".
- D3 records the block row's Done at entry, before any inlined row runs.
- D4 defines the block's Done as an inlined row reaching the exit, and no history entry or status field records that event.
- `lym loom status` therefore shows `Cartoon` done while Discussion-Write is still running.
- D4's condition also misses an exit reached by a skipped inlined row (an inlined `entry_gate` routes through its rewritten `on_done`; entry.go:51-70).
- Evidence: decision-record.md:5, 66, 77; acceptance criterion at :177.

**F3 — LOW, design. D1 makes a block impossible as the last row of a sub-recipe.**
- D1 requires `on_done` on every block row, and D2 refuses an inlined `on_done` naming a row outside its own recipe. A nested block row's `on_done` must therefore name a row of the middle recipe.
- So a middle recipe cannot end in a block, and D2's rewrite of an empty `on_done` to the exit never applies to a nested block row.
- Q9 allows nesting without stating this limit.
- Evidence: decision-record.md:32-33, 53; support-log.md:67.

**F4 — LOW, design. D3's pass-over and D3's goto onto an inlined row interact without a bound.**
- In the expanded list, the inlined rows sit before the exit, so `gotoAdmitted` admits them once the run has passed over the block. A goto there runs rows of a block whose gate did not hold, and nothing re-evaluates the gate.
- `gotoAdmitted` routes an `OutcomeSkipped` entry by `OnDone`, not by the skip target. For a skipped block row it would therefore admit the sub entry as the routed row.
- D3 bounds only the skip target, not this path.
- Evidence: goto.go:299-343 (`case Done, OutcomeSkipped: routed = def.OnDone`, :317-318); decision-record.md:68, 71.

Tool calls: none. Everything came from the inherited context.
