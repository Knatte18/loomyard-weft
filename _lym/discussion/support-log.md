# Support log — shed-sub-run-block

## Interview

### Turn 1 — exploration (autonomous)

Read the board entry, note `discussed-task-split`, task `decision-record-on-board` (peer, Discussion-Write, no record yet) and note `parts-plan-and-build`.
No `Decision for shed-sub-run-block:` note exists.
Read the recipe format and build path (`shedbuild`), the engine's producer, entry gate, status and progress (`shedengine`), the loom recipe table and accessors (`loomrecipe`), the profile template and the sites naming tapestry by name.
Findings that shaped the design: rows are addressed everywhere by name (status, goto, pause, resume, stencils), segments key the review budgets, and the loom accessors already decide by rows and engines, so build-time inlining with unchanged names makes the block invisible to most consumers.

### Turn 2 — design settled by auto-pick

No operator in autonomous mode; every question below was settled by auto-pick, recorded in the question ledger.

### Turn 3 — lens review

Six forks returned reports; none is missing.

- existing-paths, walkthrough: the block row has no registry producer and an empty engine meets engine-keyed accessors — fixed: reserved engine `Block` registered in the `Constructor` map, stamped by expansion, refused on hand-written rows (D1, D2, D3, Constraints).
- existing-paths: the block's exit needs a home after expansion so the contract check walks into the block — fixed: expansion moves `on_done` into the skip target and sets `on_done` to the sub entry (D2).
- interactions, existing-paths, walkthrough, bound-breaker: goto and relaunch entries and status read the routing projection, which carried no block path — fixed: the projection and row list carry block path and skip target (D6, D11, acceptance).
- interactions, bound-breaker: an inlined non-empty `on_done` could leave its recipe — fixed (D2, acceptance).
- interactions, leanness: a sub-recipe's `profile_recipe` was silently ignored — fixed: refused (D2, acceptance).
- interactions: nested block rows' marks contradicted D2 — fixed: a row carries the path of the blocks enclosing it (D2, D6).
- interactions, bound-breaker, walkthrough: the edge-for-edge pins could not hold — fixed: exceptions named in D9, D10 and acceptance.
- bound-breaker: D8's rationale overstated a refusal; an unprofiled `arras` task runs silently on bare config — fixed (D8 rationale).
- bound-breaker: `configreg`'s profile check reads blocks by recipe name — rejected: it validates the file's own blocks, which stay correct (existing-paths confirms).
- bound-breaker, existing-paths: `lym loom peers` row indexes differ between `arras` and tapestry — rejected: indexes are per recipe already, as darn's and tapestry's differ today.
- existing-paths: the report phase table needs a phase for `Cartoon` — fixed as a plan-writer note.
- board-scope: peer `decision-record-on-board` adds `approve_seam` to the Discussion-Bouncer `cartoon` copies — fixed: D9's pin compares whole config, Open risks names the peer.
- board-scope: "tapestry's recipe file is unchanged" fails after the peer lands — fixed: "this task makes no change to it".
- leanness: Auto-mode assumptions restated decisions — fixed: cut to picks with ids.
- leanness: the `arras` approval stated three times — fixed: dropped from D10.
- leanness: the report-grouping risk is the bench's concern — fixed: moved to the question ledger.
- leanness: the last Constraint restated tapestry unchanged — fixed: kept only the resume-identity clause.
- leanness: D6's older-status sentence repeated the tolerant-records constraint — fixed: cut.
- leanness: D7 and D11 rationales restated decisions — D7 shortened; D11 kept, as it names why one point.
- leanness: D3's skip-target validation is plan detail — rejected: an invariant the plan needs.
- leanness: the plan-writer notes may go stale — rejected: no change asked.

## Rejected alternatives

- A run-time nested shed (a second Shed driven by a block producer, sharing the status file): needs nested status, its own budget scope and its own pause and goto addressing; inlining gets the same behaviour from the existing engine.
- Prefixing inlined row names with the block (`Cartoon/Discussion-Write`): breaks every name-keyed consumer, the row-name constants in `loomshed`, stencils and the `_lym/loom/friction/<row>.md` paths.
- Routing a block's stop to a block-level `on_stuck`: rewrites the escalate-to-a-human edges every inlined row relies on, and no use asks for it.
- Writing the block gate as today's `entry_gate` setting expression: the operator asked for a named check from a shipped set, and the follow-up's check reads the board, not a setting.
- Profile blocks of its own for `arras` in `profiles.yaml`: the file is operator-owned and never reconciled, so existing hubs would refuse an `arras` task until edited by hand, and the bench wants tapestry's settings.
- Moving tapestry onto the block now, so the Discussion rows live once: the scope keeps tapestry unchanged until a bench shows no change.
- Making `cartoon` seedable as a Discussion-only run: that run stores the record on the board, which is note `discussed-task-split`'s and task `decision-record-on-board`'s.
- Other names for the new recipe: `gobelin`, `kilim`, `damask`; `arras` is short and names a fine tapestry.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 How is the block inlined? Auto-pick: build-time expansion in `shedbuild`, rows keeping their names (D2). Resolved.
- Q2 How does a goto, `lym loom status` and the progress count name a row inside a block? Auto-pick: by its own name; status shows the block beside it; the block row is a step of its own (D3, D6, D7). Resolved.
- Q3 Do a block's review segments keep their own budgets? Auto-pick: yes, by segment name, from the keys their own recipe reads (D7). Resolved.
- Q4 How do recipe keys and profiles reach a block's rows? Auto-pick: rows read the same modules; `profile_recipe` lets `arras` take tapestry's profile blocks (D8). Resolved.
- Q5 What is a block's Stuck? Auto-pick: a stop inside the block halts the parent at the inlined row; no block-level `on_stuck` (D4). Resolved.
- Q6 Where do block gate checks live, and is the gate required? Auto-pick: a closed table in `shedrecipe`; `block_gate` required; `always` ships (D5). Resolved.
- Q7 Is `cartoon` seedable? Auto-pick: no, block source only (D9). Resolved.
- Q8 The new recipe's name. Auto-pick: `arras`; awaits the operator's approval (Open risks). Open for the parent.
- Q9 May blocks nest? Auto-pick: yes, with a self-inclusion refused and a block path mark; it costs a stack in the expander (D2). Resolved.
- Q10 Do reports mix tapestry and `arras` runs? A report grouping runs by row name and not recipe would, since both share row names; the bench follow-up should group by recipe. Noted for the bench, out of this task.
