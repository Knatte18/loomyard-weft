```yaml
lens: bound-breaker
outcome: reported
```

**Lens: bound-breaker.** I attacked each bound, guard, guarantee and claim about existing code in the decision record. Four are broken. The rest held.

**Claims attacked**
- D1, Parse refuses a hand-written `engine: Block` while Expand stamps one: held. `shedbuild.NewShedFrom` does not run `checkRecipeShape` again, so the expanded recipe never meets the new refusal (`internal/shedbuild/parse.go:67`, `newshed.go:83`).
- D2, the inlined-edge refusal and the empty-`on_done` rewrite under nesting: **broken** (F1).
- D2 bound, the same recipe used by two blocks is refused by the row-name check: held.
- D2, the segment refusal keeps blocks from sharing a bounce episode: held. Episodes key on segment (`progress.go`).
- D3 bound, only a block row carries a skip target and only its gate's not holding takes it: **broken** in the goto admission reader (F4).
- D3, a goto onto the block row or an inlined row: the bypass has no stated bound (F3).
- D4, the block's Done condition: **broken** for a skipped last row (F5).
- D6, an older status file still reads: held (`ParseStatusTolerant`). Outside shedengine, only seed writers build history, and they write it empty (`loomshed/seed.go:58`, `battencli/arm.go:780`). D6 holds.
- D7, a block's Discussion-Review reads the discussion perch keys: held. The perch table is keyed by segment name (`internal/loomcli/wiring.go:1069`).
- D7, a run that passes over a block jumps straight to its exit's step: held (`mainLine`, `stepIndex`).
- D8 rationale, an unprofiled `arras` task runs on bare config and a named profile is refused: held (`runprofile.Settings`, `Default`; `battencli/runsettings.go:28-37`).
- D8, every profile-block reader goes through one accessor: held. `battencli/runsettings.go` is the only reader keyed by recipe, and `configreg.checkProfiles` reads only the file's own blocks.
- D8 bound, "it changes no row, budget or config module": **broken** (F2).
- D9, `cartoon` cannot be seeded as a run: held. Seed-Child calls `shedrun.ValidateRecipe` (`internal/battencli/wire.go:1392`), and `childRecipeSupported` admits only its listed recipes.
- Open risk on the peer's `approve_seam` change, "whichever lands second carries it into `cartoon`": held in both landing orders, because D9's config pin compares against tapestry's live row.
- D10, `arras`'s top-level keys equal tapestry's, and its expanded Discussion-Bouncer `on_done` becomes Drift-Baseline as in tapestry: held.
- D11, batten and pawl tell no resolver: held.

**Findings**

**F1 — MEDIUM, design. D2 (`decision-record.md:44`, `:53`; D1 `:32`; support log Q9 `:67`).**
D2 does not say whether the "edge leaves its own recipe" refusal reads an inlined row's source edges or its expanded ones. Read on expanded edges, it refuses every nested block, which contradicts Q9 ("blocks may nest").
- Counterexample: outer recipe P has a block on recipe M, and M has a block on recipe I.
- After I is expanded into M, each I row whose empty `on_done` was rewritten now names M's exit row, which lies outside I.
- M's inner block row now has `on_done` set to I's entry, which lies outside M.
- Both trip the refusal at `:53`.

A second break: D1 requires every block row to carry `on_done`, and D2 refuses an `on_done` naming a row outside its recipe. So no sub-recipe can end with a block row.

The fix: state the expansion order (innermost first, or outermost first), and state that the check reads the source edges with the rewritten ones exempt.

**F2 — LOW, consistency. D8 bound (`decision-record.md:114`).**
The bound says `profile_recipe` "changes no row, budget or config module". But the tapestry blocks of the shipped profiles set the review budgets (`internal/runprofile/template.yaml:18`, `:27`, `:37`: `discussion_max_bounces`, `plan_max_bounces`, `code_max_bounces` and their hard caps). `profile_recipe: tapestry` applies exactly those to an `arras` task.

The bound should say what the key lets through: it chooses which recipe's profile block, budget keys included, the task's settings come from. What limits it: only a top-level key of an embedded recipe sets it, and Expand refuses it on a sub-recipe.

**F3 — MEDIUM, design. D3 attack surface (`decision-record.md:68`, `:71`).**
D3 makes the block gate bypassable by goto, onto the block row or any inlined row, but states no bound for that bypass. Its only `Bound:` line covers the skip target.

Counterexample: a block was passed over (the follow-up's discussed-entry gate), and the run now stands at Plan-Write.
- `gotoAdmitted` admits every row before the current one in list order (`internal/shedengine/goto.go:334-341`).
- The inlined rows sit before the block's exit in that order, so `lym loom goto --to Discussion-Write` is admitted.
- Discussion then runs inside a block whose gate said to pass over it, and it rewrites the record the gate relied on.

D3 also says "as a goto bypasses any target's gate", but a Relaunch onto the block row clears the entered row and so evaluates the gate (`goto.go:243-244`). D3 does not state that difference.

The bound to state: only a goto bypasses the gate; goto only moves a halted run back; only the parent or operator issues it; and Relaunch evaluates the gate again.

**F4 — LOW, design. D3 skip-target bound (`decision-record.md:65`, `:71`).**
`gotoAdmitted` takes the routed row of a skipped entry from `def.OnDone` (`internal/shedengine/goto.go:317-318`). On an expanded block row, `OnDone` is the sub-recipe's entry, not the skip target.

Counterexample: a block is passed over, and the status's current producer names no row, for example a retired name. Goto then admits the block's first inlined row as the "routed" row, though the run never went there.

D3 says a skipped row routes by its skip target, but the record applies that only to the skip itself (the plan-writer note names `entry.go`). The fix: state that every reader of a skipped entry's route uses the skip target.

**F5 — LOW, consistency. D4 Done condition (`decision-record.md:77`).**
D4 says "the block is Done when an inlined row whose expanded `on_done` is the block's exit returns Done". It misses one path: an inlined row with an `entry_gate`, whose `on_done` is the exit, can be skipped. A skip routes by `OnDone` as a Done does (`internal/shedengine/entry.go:50-70`), so the block ends with no Done recorded.

`cartoon` has no gated row today, so this is latent. Fix: "returns Done or is skipped".

**F6 — NIT, consistency. D9 (`decision-record.md:122`) against D2 (`:56`).**
Parse refuses a recipe whose `terminals` is empty (`internal/shedbuild/parse.go:83-85`), so `cartoon` must list one. D2 says a sub-recipe's `terminals` are never read, and D9 does not say what `cartoon`'s should name.

**Tool calls**
1. A single grep across `newshed.go`, the `ValidateRecipe` callers, the loom segment-to-perch table, `runprofile/template.yaml` and the `HistoryEntry` writers outside shedengine. My context lacked whether `NewShedFrom` checks the recipe shape again, how board recipes are validated, how the segment table is keyed, what the shipped profile blocks set, and which other packages write history.
