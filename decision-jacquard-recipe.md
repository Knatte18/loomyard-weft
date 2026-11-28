## Goal

Jacquard recipe holds tapestry's rows after Discussion, tapestry becomes the Motif and Jacquard blocks, and a Record-Seed row seeds the stored record

Takes: D1 whole: contracts/recipes/jacquard-recipe.yaml holding Preflight, Record-Seed and every tapestry row from Drift-Baseline to Friction-Reflect, moved with routing, segments, gates and comments, never copied, with jacquard's top-level keys; shedrun's recipe-name constant jacquard in the closed vocabulary (RecipeNames, OfferedRecipeNames, ValidateRecipe) and shedcli's recipe table kept in step with it; the embed in contracts/recipes/recipes.go; internal/loomrecipe's embedded-recipe map and the loomrecipe recipe lists jacquard's own rows need to run alone and as a block (RecipeEngines, bouncerrows.go, finalizerow.go). D2 whole: tapestry-recipe.yaml as the Motif block row (block_gate undiscussed, on_done Jacquard) and the Jacquard block row (block_gate always, no on_done), entry Motif, row names unchanged so an in-flight tapestry run resumes by row name; the tapestry and motif recipe headers rewritten for the move and naming jacquard, which is D10's recipe-header step. D3 whole: shedbuild.Expand admits a last outermost block row with empty on_done and block_gate always, refuses a last one with another gate with the way forward "set on_done, or use block_gate always", and refuses every other outermost empty on_done as today. D4's engine and contract steps: the RecordSeed registry entry in internal/shedrecipe, the RecordSeed producer in internal/loomshed (Record-Write's stored-only mode, writing only a missing record or support log and committing the pair, Stuck with nothing written when the record is missing and no stored record exists, its reason naming the tapestry goto, the jacquard batten sequence and the jacquard re-seed with the seed's path, driver and parent filled in as disagreeingSeedRefusal does), its Env wiring in internal/loomcli/wiring.go and testkit/envkit, the widened loom contract in internal/shedbuild/loomcontract.go (at most one record row, at most one seed row, at least one, seed after record on the on_done path, the seed read as the record row when no record row exists), and loomrecipe.RecordEngine returning RecordSeed for a recipe with only a seed row. Per the added entry, no plan or build recipe, pair handover or Worktree-Teardown skip.
Delivers: A jacquard recipe that builds as a loom recipe from Preflight to Friction-Reflect; tapestry expanded as Motif's rows followed by Jacquard's with the same edges; a tapestry run of an undiscussed entry walking Motif, Discussion, Preflight, Record-Seed (writing nothing) and Drift-Baseline; Record-Seed seeding, keeping or Stuck per D4; the loom contract building tapestry, jacquard, darn and motif under the widened clauses.
Exports: Fixed for later parts: shedrun.RecipeJacquard ("jacquard"); the engine name RecordSeed in shedrecipe's registry and beside LoomRecordEngines; loomshed.NameRecordSeed ("Record-Seed"); the block row name Jacquard in tapestry-recipe.yaml; loomrecipe.RecordEngine returning DiscussionWrite for tapestry and RecordSeed for jacquard; loomrecipe.EngineRow unchanged in signature.

## Scope

The part changes these packages:

- contracts/recipes (about 490 changed lines)
- internal/shedrun (about 20 changed lines)
- internal/shedcli (about 5 changed lines)
- internal/shedrecipe (about 110 changed lines)
- internal/shedbuild (about 190 changed lines)
- internal/loomshed (about 270 changed lines)
- internal/loomrecipe (about 190 changed lines)
- internal/loomcli (about 40 changed lines)
- internal/testkit/envkit (about 10 changed lines)

## Decisions

### D1 — Jacquard is its own recipe file holding tapestry's post-Discussion rows

`contracts/recipes/jacquard-recipe.yaml` (new) holds Preflight, then Record-Seed (D4), then every row tapestry's file holds today after its Motif row, from Drift-Baseline to Friction-Reflect.
Those rows move with their routing, segments, gates and comments, never copied.
Its top-level keys are `entry: Preflight`, `amendment_target: Plan-Burler-Review`, `contract: loom`, `verify_source: plan` and `terminals: [Friction-Reflect]`.
`shedrun` gains the recipe-name constant `jacquard` in its closed vocabulary, `contracts/recipes/recipes.go` embeds the file, and `internal/loomrecipe`'s embedded-recipe map resolves it, so it runs alone and as a block.
The recipe is spelled `jacquard` wherever config, seeds, profiles and records name a recipe.
Its rows read the same `tapestry.*` and `loom.*` config keys they read today.

Rationale: one source for the build rows, and `run jacquard` names a run in which Discussion never runs.

### D2 — Tapestry is the Motif block followed by the Jacquard block

`tapestry-recipe.yaml`'s rows become two block rows:

- `Motif`: `block: motif`, `block_gate: undiscussed`, `on_done: Jacquard`, as today but for its exit.
- `Jacquard`: `block: jacquard`, `block_gate: always`, no `on_done`.

Its `entry` becomes `Motif`.
It keeps its own top-level keys, since an expansion reads none of a block's; all but `entry` equal jacquard's.
Preflight now runs after Discussion in a tapestry run.

Row names are unchanged, and a tapestry run started before this change resumes under the new graph by row name.
A run parked inside the Motif block goes on to Preflight and Record-Seed, which finds the approved record and writes nothing.
A run parked at Preflight that never entered Motif goes on to Record-Seed:
with a stored record it seeds it, which is what the Motif gate would have done, and with none Record-Seed stops it per D4.
The rows from Preflight on carry block path `Jacquard`, so the run checklist shows them, and the Plan-Review and Code-Review segments, under the Jacquard block.
History entries an in-flight run wrote before the change carry no block; the checklist joins them to their rows by name, so only their block label differs.

Rationale: tapestry's rows are motif's plus jacquard's with no copy.
Preflight's checks other than its worktree and fabric checks guard config only the build rows read;
those two also cover Discussion's commit, which motif already runs without,
so a fabric fault in an undiscussed tapestry run now shows at Discussion first.
A Preflight halt after Discussion keeps the approved record.

### D3 — A last outermost block row may end the run

`shedbuild.Expand` admits an outermost block row with an empty `on_done` when it is the last row of its recipe and its `block_gate` is `always` (`shedrecipe.BlockGateAlways`).
The rows it inlines keep an empty `on_done` where they set none, so the row that ends the sub-recipe run alone ends the parent run.
A last outermost block row with an empty `on_done` and any other gate is refused, with the way forward "set on_done, or use block_gate always",
since its skip would fall back to the block's own entry.
Every other outermost block row with an empty `on_done` is refused as today.
A nested block row's empty `on_done` still takes the exit of the block enclosing it.

Bound: the edge lets an empty `on_done` end the run only inside a block that is the parent's last row.
That block's gate always holds, so no skip can reach the empty exit.
For a loom recipe the loom contract's terminal check still requires Finalize to be followed only by terminals.

Rationale: tapestry's Jacquard block has no row after it, and Friction-Reflect's empty `on_done` is what ends a run quietly.

### D4 — A Record-Seed engine seeds the stored record, and the loom contract admits it

A new registry engine, `RecordSeed`, runs as row `Record-Seed`.
It is Record-Write's producer in a stored-only mode: it writes whichever of the decision record and support log is missing, the record from the entry's stored record, and commits the pair.
A record that exists is never overwritten, so an approved record and every entry added after Discussion survive.
With the record missing and no stored record, the row is Stuck with nothing written.
Its reason names a way forward per recipe:

- a jacquard run names two cases, as `lym loom start`'s refusal does (D5):
  - a pair batten made: `lym batten cancel <slug> --remove` from the hub's prime worktree, `lym board upsert '{"slug":"<slug>","recipe":"tapestry"}'`, `lym board set-status '{"slug":"<slug>","status":null}'`, then `lym batten run <slug>`;
  - any other pair, from its task worktree: `rm <seed path>`, `lym shed seed self --recipe tapestry --driver <driver> --param parent=<parent>`, `lym loom goto --to Discussion-Write`, `lym loom start`, then `lym loom resume`;
    the reason fills in the seed's path and its recorded driver and parent, as `disagreeingSeedRefusal` does,
    and the start commits the new seed before the resume lifts the goto's pause;
- a tapestry run: `lym loom goto --to Discussion-Write`, then `lym loom resume`.

The loom contract in `internal/shedbuild/loomcontract.go` changes from "one record row" to:
a loom recipe holds at most one record row (DiscussionWrite or RecordWrite) and at most one seed row (RecordSeed), and at least one of the two.
When it holds both, the seed row follows the record row on the on_done path from the entry.
Every other clause reads the seed row as the record row when the recipe holds no record row.
`loomrecipe.RecordEngine` keeps returning the record row's engine, so tapestry's decision verbs read Discussion as today, and returns RecordSeed for a recipe with only a seed row.
The decision verbs treat RecordSeed as they treat RecordWrite, with Record-Seed among the rows before the record is written.
The RecordWrite-path refusals in `internal/loomcli/decision.go` (`decisionWayRecordStart`, `decisionWayRecordResume`, `decisionWayRecordGoto`, `decisionWayUncommittedGoto`) and `recordWritePending` name the row of the run's recipe that runs the record or seed engine,
read from the recipe's rows through `loomrecipe.EngineRow`, never the literal Record-Write.
Darn's Record-Write row is unchanged.

Bound: a seed row writes only from the stored record, never from the entry body, and the contract admits it only beside or in place of the one record row.

Rationale: jacquard needs the seed tapestry's Motif gate gives a discussed entry, and the contract needs one row to name the record's source.
Writing only a missing file makes the row a no-op after tapestry's Discussion, so the seed never clobbers the run's record.

### Added after Discussion (arbiter, 2026-11-28): Jacquard stays one recipe of every row after Discussion; the plan-and-build-recipes note is later work

Decision: Keep D1 as written: jacquard is one recipe holding Preflight, Record-Seed and every tapestry row from Drift-Baseline to Friction-Reflect, and tapestry is the Motif block followed by the Jacquard block (D2). The board note `plan-and-build-recipes` (F3, project `plan-ahead-chains`), which splits those rows into a plan recipe and a build recipe, is not designed or built in this run. It stays a note for later work that may split jacquard. The record's Scope Out list gains no entry for it, since it names no part of this task; no card of this run adds a plan or build recipe, a Y-to-Z pair handover, or a Worktree-Teardown skip.

Rationale: The task's board entry settles it. Its Idea has jacquard hold 'Plan, Plan Perch, Webster, review and landing', and its operator-marked 'Tapestry stays (operator, 2026-11-28)' passage has tapestry made of motif followed by jacquard. The note is a separate, unpromoted note; building its split here would widen the task's scope with new recipes, pair handling and batten teardown changes the entry never asks for. Answers review finding F8 of round 1.

### Added after Discussion (arbiter, 2026-11-28): The jacq nickname is written by the run's parent after landing, as D11 says

Decision: Keep D11 and its Open risks line unchanged. No card of this run writes the 'jacq' nickname or Bolt's `heddle.json`. Once the run has landed, the run's parent adds the nickname line D11 gives to the `nicknames` entry with `lym heddle set`, which completes the board entry's ask that the task add 'jacq' to the repository's nicknames. The Review criterion for D11 stays: no file of the diff writes Bolt's `heddle.json` or a nickname line.

Rationale: `PATTERN-heddle-instructions` has `heddle.json`, which lives at the Bolt checkout's root and not in this worktree, written only through the `lym heddle` verbs, and `lym heddle set` renders into every live worktree of the hub at once. A card writing it before landing would publish a nickname for a recipe the hub's binary cannot run yet, the alternative the support log rejects. The board entry asks only that the nickname be added, not by which step, so D11's route answers it. Answers review finding F9 of round 1.

### Added after Discussion (parent, 2026-11-28): Nickname jacq written after landing

Decision: Accept D11: the parent writes the jacq nickname into the heddle nicknames entry with lym heddle set once the run lands; no card of this run writes it.

Rationale: The heddle glossary is hub-level text the orch keeps; writing it after landing keeps the run's scope to code and avoids a nickname for a recipe that has not landed.

### Added after Discussion (parent, 2026-11-28): Jacquard gains no motif-run clear; a Sketch goes only to raddle

Decision: Apply F16's fix: delete D5's motif-run passage. A finished motif run turns its entry into a Sketch (finishesAsSketch in internal/loomcli/wiring.go), which only a raddle run takes (checkDiscussedPromotion in internal/boardengine/sketch.go), so jacquard builds the parts raddle files and other discussed tasks, and batten gains no motif-run clear for it. D5's Bound drops the clause on reading the slug's done run for the clear; the acceptance criterion on a jacquard start after a done motif run and the Constraints fact on a motif run's task branch are dropped. Batten's refusal keeps its lym batten cleanup --apply clause for a task whose Sketch the board refused. F17 falls with the passage it concerned; nothing else changes. Answers round-3 findings F16 and F17.

Rationale: The board's standing rule, kept by the operator's board policy, is that a Sketch is split only by its raddle task; the entry asks for jacquard on raddle's parts, not for a Sketch to run jacquard directly. The fix narrows D5 to that and removes a recovery sequence that loops on two refusals.

## Constraints

- `PATTERN-shed-recipe-registry`: jacquard declares `contract: loom` in its standard form; the start refusal reads rows, not the name; RecordSeed is one row of the registry map.
- `PATTERN-code-conventions`: the recipe name is one named constant in `shedrun`; the refusal fills it from that constant.
- `PATTERN-refusal-way-forward`: each start refusal site, Record-Seed's Stuck and the decision verbs' RecordWrite-path refusals name their ways forward, and each lands with a test that reaches it.
- `PATTERN-config-layers`: no config key moves; the new profile blocks are template additions the hub reconcile carries, and D7's one-time reinterpretation covers the cases the reconcile leaves.
- `PATTERN-heddle-instructions`: the nickname changes only through `lym heddle set`.
- Fact, from `internal/shedbuild/expand.go`: a block's rows keep their names, segments must not clash with rows outside, and an outermost block row with an empty `on_done` is refused today.
- Fact, from `internal/shedbuild/loomcontract.go`: the loom contract today refuses a recipe with other than one DiscussionWrite or RecordWrite row.
- Fact, from `internal/loomshed/recordwrite.go`: Record-Write writes only a missing record or support log and commits every call.
- Fact, from `internal/boardengine/raddlefile.go` and `internal/loomshed/raddlefile.go`: filing writes each part's record, sets its discussed flag and re-runs the proposal check first.
- Fact, from `internal/runprofile/doc.go`: the reconcile gives a profiles file each template key it holds as its base does, gained ones included.
- Fact, from `internal/loomcli/arm.go` (`resolveRecipe`), `internal/shedengine/status.go` and `internal/loomcli/sharedbootstrap.go`:
  every loom verb reads its recipe from the seed and the status file names none,
  so a re-seeded run goes on under the new recipe from its current row,
  and a `lym loom start` over an existing status file commits the new seed and wakes the driver.

## Auto-mode assumptions

- Config keys stay under `tapestry.*` (D1).
- Preflight runs after Discussion in tapestry (D2).
- The seed is its own engine under a widened loom contract (D4).
- Raddle's `tapestry` shape keeps its name (Scope).
- A jacquard profile block is tapestry's minus `loom.discussion` (D7).
- The nickname is written by the parent after landing (D11).

## Open risks

## Acceptance criteria

- D1, D2: the expanded tapestry recipe's rows equal the Motif block's rows followed by the Jacquard block's rows, in order with the same edges, and jacquard's top-level keys other than `entry` equal tapestry's (tier 0, `internal/loomrecipe`).
- D1: `jacquard` is in the recipe-name vocabulary and builds as a loom recipe with Preflight as its entry and Friction-Reflect as its terminal (tier 0).
- D2: a tapestry run of an undiscussed entry walks Motif, Discussion, Preflight, Record-Seed with nothing written, then Drift-Baseline; one of a discussed entry passes over Motif with the record seeded and Record-Seed writes nothing (tier 0, a recipe walk over the real rows).
- D3: Expand admits a last outermost block row with no `on_done`, whose inlined terminal row keeps an empty `on_done`; it still refuses an outermost block row with no `on_done` that is not last, and refuses a last block row with no `on_done` whose gate is not `always` (tier 0, `internal/shedbuild`).
- D4: Record-Seed seeds a missing record from the stored record, leaves an existing record untouched, and with no stored record is Stuck with nothing written;
  its reason names, per case, the tapestry goto, the jacquard batten sequence and the jacquard re-seed for a pair batten did not make (tier 0, `internal/loomshed`).
- D4: the loom contract builds tapestry, jacquard, darn and motif, and refuses a recipe with two record rows, two seed rows, neither, or a seed row ahead of its record row (tier 0, `internal/shedbuild`).
