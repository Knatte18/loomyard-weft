```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism end to end on `arras` (Preflight → Cartoon → cartoon's rows → Drift-Baseline …) and the code paths in context.

**Findings**

1. MEDIUM — D6 / D11: the goto entry gets no block mark because the routing projection carries no block path.
   - Evidence: `lym loom goto` builds `shedengine.GotoRequest` from `spec.Routing.Producers` (`internal/shedverbs/goto.go`), a projection with no engines built.
   - Goto then appends `HistoryEntry{Producer: req.Target, Outcome: OutcomeGoto}` (`internal/shedengine/goto.go`).
   - D6 says a goto entry for an inlined row carries the block path, and that `lym loom status` shows it. Status also reads only the projection.
   - D11 adds "a routing projection from an already-expanded recipe" but never says the projected `ProducerDef` carries the block path or the skip target. Today's `RoutingOf` copies only Name, OnDone, OnStuck, OnReroute, Segment and MaxBounces.
   - As written, the goto entry and the status display get an empty block.
   - Fix: D11 should state that the projection carries the block path and the skip target.

2. MEDIUM — D3 against `PATTERN-shed-recipe-registry`: the block row's producer has no registry entry.
   - Evidence: `Build` calls `shedrecipe.Lookup(row.Engine)` for every row, and the PATTERN entry says every producer comes from the one `Constructor` map.
   - D3 gives the block row "a producer [that] returns Done at once", but D1 forbids `engine` on a block row and D5 adds only a gate-check table.
   - So this producer would be built in `shedbuild` or `shedengine`, outside the registry. The PATTERN change in the record does not cover that.
   - Fix: register the producer as a `Constructor` entry under a reserved engine name that Parse refuses on hand-written rows, or name the exception in the PATTERN change.

3. LOW — D1 / D11: a block row with no engine meets code that reads engines.
   - Evidence: `checkRowShape` refuses an empty `engine`, and Parse runs it on every row.
   - The expanded recipe keeps the block row with an empty engine. `RecipeEngines` and the coverage guard built on it would then gather `""` if they read `arras`. `HasEngine` and the other row loops in `recipekeys.go` would meet it too.
   - Nothing in the record says these skip block rows.
   - Fix: state that block rows are left out of engine sets, or give them the reserved engine from finding 2.

4. LOW — Acceptance criterion for D10 ("edge for edge"): a literal test of it fails.
   - Evidence: tapestry's Preflight goes `on_done: Discussion-Write` and its Discussion-Bouncer goes to Drift-Baseline.
   - In expanded `arras`, Preflight goes to Cartoon, Cartoon's Done goes to Discussion-Write, and the inlined Bouncer's Done is rewritten to Drift-Baseline.
   - Preflight's edge differs, so the criterion has to name that exception: Preflight → Cartoon → Discussion-Write in place of Preflight → Discussion-Write.

**Each decision: case used, purpose met or not**

- D1 — `arras` Cartoon row with `block`, `block_gate` and `on_done`, and a row that also sets `engine`: met. The empty-engine check in `checkRowShape` must exempt block rows (plan detail, part of finding 3).
- D2 — expanding `cartoon` into `arras`: the Bouncer's empty `on_done` becomes Drift-Baseline, the Burler rows' non-empty edges stay, and there is no name or segment collision. Met.
- D3, gate holds — `always`: the Cartoon producer returns Done, the history records "Cartoon done", and the run goes to Discussion-Write. Met, apart from the registry gap in finding 2.
- D3, gate does not hold — skipped entry, then the skip target, then Drift-Baseline: met. `entry.go`'s `routedState`, `persistRouted(def.OnDone)` and `findProducer(def.OnDone)` all switch to the skip target (plan detail).
- D3, goto onto Cartoon: the gate is bypassed and the block is entered. Met.
- D4 — Discussion-Write returns Stuck with no `on_stuck`: the run halts blocked at Discussion-Write, and a resume calls it again. Met.
- D5 — `block_gate: always` builds a gate that always holds, and an unknown name is refused. Met.
- D6 — a shed-appended entry for an inlined row (`run.go`, `entry.go`, `reroute.go` all hold the def): met. A goto entry: not met (finding 1).
- D7 — progress on `arras`: Preflight 1, Cartoon 2, Discussion-Write 3, Discussion-Review 4, Drift-Baseline 5 …; the budget is keyed by segment through `loomSegmentPerches`. Met, provided `reviewSegments(source)` in `Routing` also reads the expanded recipe, which D11's "every accessor" covers.
- D8 — Seed-Child's `prepareRunSettings` in `internal/battencli/runsettings.go` reads `profiles.Default(settings.Recipe)` and `profiles.Settings(name, settings.Recipe)`. battencli is wiring and can resolve `profile_recipe` through `loomrecipe`. Met.
- D9 — `cartoon`'s rows against tapestry's Discussion rows, with the Bouncer's `on_done` excepted as the record states. Met.
- D10 — a board task with recipe `arras`: seeded through `shedrun.RecipeNames`, batten's supported list, `loomrecipe.New` with expansion, `withTapestry` through `Rows` (Plan-Review is present), and `RecordEngine` (DiscussionWrite). Met, apart from the criterion wording in finding 4.
- D11 — the build, `Routing`, `Rows`, `HasEngine` and `GateFixerSetting` on `arras` give the same expanded rows. Met, apart from the projection fields in finding 1.

**Tool calls**

1. grep for `runprofile.` callers — to find who reads a task's profile block for D8; not in context.
2. grep of recipe use in the runsettings files, plus a `go list -deps` check — to confirm Seed-Child's profile lookup goes by recipe name and to check for an import cycle; not in context.
3. grep of the `HistoryEntry{` append sites and `GotoRequest` builders — to trace whether a goto reads the built shed or the projection for D6; not in context.
