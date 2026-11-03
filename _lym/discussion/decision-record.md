# Decision record — a shed block runs a whole sub-recipe inline

## Goal

A recipe row of a new kind, a block, names another recipe and runs that recipe's rows inside the parent run, entered as one step that ends when the run reaches the block's exit.
A block carries an entry gate, a check with no agent from a set lym ships, that enters the block or passes over it whole.
The block is proven on a real recipe: `cartoon` holds tapestry's Discussion rows, and a new recipe beside tapestry, `arras`, runs `cartoon` through a block and then tapestry's Plan and build rows.
Tapestry itself stays unchanged.

## Scope

In:

- The block row in the recipe format, its inline expansion in `internal/shedbuild`, and its run-time behaviour in `internal/shedengine`: entry, pass-over, history marks, goto, pause, progress and review budgets.
- The set of block gate checks lym ships, with `always`, which always enters.
- `cartoon` as a recipe file of its own holding tapestry's Discussion rows, usable as a block.
- `arras`, a seedable loom recipe beside tapestry: Preflight, a `cartoon` block, then tapestry's rows from Drift-Baseline on.
- `lym loom status` and the progress count naming a row inside a block.

Out:

- The gate check that passes over `cartoon` for an entry already discussed, reading the record from Bolt into `_lym/discussion`: a follow-up that waits on this task and on task `decision-record-on-board`.
- The bench of `arras` against tapestry, and moving tapestry onto the block.
- `cartoon` as a run of its own (the Discussion-only run of note `discussed-task-split`); here it is a block source only.
- A sub-run in another worktree, which stays `battenshed.NewInnerRun`'s kind.

## Decisions

### D1 — A block row in the recipe format

A producer list entry may be a block row instead of an engine row.
A block row carries exactly `name`, `block` (the name of the recipe it runs), `block_gate` (the name of its gate check) and `on_done` (where the run goes once the block ends or is passed over).
Parse refuses a block row that also sets `engine`, `config`, `on_stuck`, `on_reroute`, `segment`, `max_bounces` or `entry_gate`, and one missing `block` or `block_gate`, each naming the row and the way forward.
A block row's `on_done` may be empty: inside another block, expansion rewrites it like any inlined row's (D2), so a sub-recipe may end in a block; Expand refuses it empty on a block row outside every block.
Parse also refuses a hand-written row whose `engine` is `Block`, the engine D3 reserves for block rows.
`block_gate` is required, so a reader of the recipe sees whether the block can be passed over.

Rationale: the block is declared in the recipe file, so a recipe composes others with no code per recipe; a block has no engine config, budget or segment of its own because its rows carry theirs.

### D2 — Inline expansion in shedbuild

`shedbuild.Expand` (new) takes a parsed recipe and a told resolver from a recipe name to its bytes, and returns the recipe with each block row followed by the rows of the recipe it names.
On the block row, expansion sets the engine `Block`, moves its `on_done` into a skip target the row carries outside the YAML format, and sets its `on_done` to the sub-recipe's `entry`; so the contract check, the routing projection and the build all walk the block row's Done into the block.
Rows keep their own names, so a row inside a block is addressed, recorded, paused and gone to by the same name as in its own recipe.
Expansion runs innermost block first: a block's recipe is expanded, its own blocks included, before its rows are inlined into the parent.
A block's exit is its block row's `on_done` as its recipe file sets it, or, when that is empty, the exit of the block enclosing it.
Expansion rewrites every empty `on_done` of an inlined row, and the empty skip target of an inlined block row, to the exit of the innermost block enclosing that row, so the sub-recipe's end reaches the block's exit.
Each inlined row carries the path of the blocks that enclose it, outermost first, joined by `/`; a row outside every block, and so the outermost block row, carries none.

Expand refuses, each naming the block and the way forward:

- a block naming a recipe the resolver does not know, and any block row when the resolver is nil;
- a block whose recipe, directly or through its own blocks, names itself;
- an inlined row whose name equals a row outside its block;
- an inlined row whose `segment` is also used by a row outside its block, so two blocks never share a bounce episode or budget;
- an inlined row whose `on_done`, `on_stuck` or `on_reroute`, as its own recipe file sets it, is non-empty and names a row outside that recipe; an edge expansion itself sets, a rewritten empty `on_done` or skip target or a block row's `on_done` set to its sub-recipe's entry, is exempt;
- a block row outside every block with an empty `on_done`;
- a block row whose `on_done` names the block row itself or a row its own expansion inlines;
- a sub-recipe that sets `contract`, `verify_source`, `amendment_target` or `profile_recipe` (D8), which belong to the run, not to a block.

A sub-recipe's `terminals` are not read when it is inlined; the parent's are the run's.
Bound: the same recipe used by two blocks in one parent is refused by the row-name check, which is accepted until a use needs it.

Rationale: inlining at build time gives one run, one status file and one driver, so pause, goto, review budgets and resume work unchanged on the inlined rows.
Expansion lives beside `shedbuild.Parse`, the sole recipe parser, and the resolver is told, so `shedbuild` stays recipe-agnostic.

### D3 — A block at run time

`Block` is an entry of the producer registry in `internal/shedrecipe`, whose producer returns Done at once; Build builds the block row through it like any row.
`shedengine.ProducerDef` gains the skip target D2 sets: a skipped row routes to it when it is set, and by `OnDone` otherwise, so every existing row skips as today.
Every reader that reconstructs where a skipped entry routed, the goto admission's routed row included, reads the skip target the same way.
A block gate that holds enters the block: the block row records Done, which marks entering the block and not its end, and the run goes on to the sub-recipe's entry; `lym loom status` shows a block row's Done entry as entered.
A gate that does not hold passes over the block whole: the block row records a skipped entry with the gate's reason, and the run goes to the skip target, so no inlined row runs or is recorded.
A goto onto the block row enters the block without evaluating its gate, as a goto bypasses any target's gate, while a re-launch onto it clears the entered row and evaluates the gate; a goto or re-launch onto an inlined row moves the run there as onto any row.
Validation refuses a skip target that names no producer.

Bound: the skip target lets the run jump past every inlined row; only a block row carries one, and only its gate's not holding takes it.
Bound: only a goto or a driver's re-launch bypasses the block gate; both move only a halted run, and only to a row `CheckGotoTarget` admits.
A move onto an inlined row of a passed-over block runs that row with the block gate never held, which is the mover's decision; a re-launch onto the block row clears the entered row and evaluates the gate again.

Rationale: the block row gives the gate a place to run and the history a place to record entering or passing over, with the entry gate machinery shedengine already has and a producer from the one registry.

### D4 — A block's end and Stuck

The block ends when the run reaches its exit by a Done or a skip: an inlined row whose expanded `on_done` is the exit returns Done or is skipped by its entry gate, or an inlined block row whose skip target is the exit is passed over.
The end has no history entry of its own; a reader finds it as the exit's first entry after the block row's latest Done entry, so a goto out of the block or a pass-over of it never reads as its end.
A stop inside the block halts the parent run at the inlined row that stopped, with that row's reason, as the same row halts its own recipe's run today; the block row takes no `on_stuck` (D1), so a block's Stuck is the parent run's halt.
Resume, `lym loom goto` and the review rings inside the block then work as in the sub-recipe's own run.

Rationale: inline, the sub-run's stop and the parent's halt are one event; routing a stop elsewhere would change the halts every inlined row relies on.

### D5 — Block gate checks lym ships

The block gate checks are a closed table in `internal/shedrecipe` keyed by check name, each entry building the block's `shedengine.EntryGate` from the told `shedrecipe.Env`, beside the producer registry.
This task ships one check, `always`, which always enters.
Build refuses a `block_gate` that names no check in the table, naming the row and the checks there are.
The follow-up's check that passes over `cartoon` for a discussed entry is a new row of this table.

Rationale: the operator wants the gate named in the recipe from a set lym ships, never code per recipe; one table keyed by name keeps that set closed and testable.

### D6 — History marks a row's block

`shedengine.HistoryEntry` gains a `block` field, omitted when empty, and `shedengine.ProducerDef` gains the block path D2 sets.
Every history entry for an inlined row carries that row's block path: those the shed appends, skipped entries included, and goto and relaunch entries, which read it from the routing projection (D11).
`lym loom status` shows an inlined row's block beside its name, in its row list and in its history lines.

Rationale: the operator asked for history entries marked with their block; the mark is additive, so a tolerant reader of an older record still reads it.

### D7 — Progress and review budgets inside a block

The progress count and the status pane's entry numbers treat the block row as a step of its own, like any mechanical row, and the inlined rows as the steps they are in their own recipe, a review segment collapsing to one step as today.
A run that passes over a block moves straight from the block row's step to the step of its exit.
A review segment inside a block keeps its own name, so its bounce budget, hard cap and perch keys are the ones its own recipe reads: the Discussion-Review segment of `cartoon` reads `loom.yaml`'s discussion perch keys exactly as tapestry's does.

Rationale: unchanged names and segments let every existing projection and budget key serve the block.

### D8 — Settings and profiles reach a block's rows

An inlined row reads the same config modules and keys as in its own recipe; a block adds no settings layer.
A recipe may set a new top-level key, `profile_recipe`, naming the recipe whose `profiles.yaml` blocks and default profile it takes; empty means its own name.
`arras` sets `profile_recipe: tapestry`, so an `arras` task takes the same profile and keys as a tapestry task, and a hub's existing `profiles.yaml`, which no reconcile rewrites, serves `arras` with no edit.
Every reader of a task's profile block resolves the recipe name through that key in one accessor of `internal/loomrecipe`.
Bound: `profile_recipe` chooses which recipe's `profiles.yaml` block and default profile, review budget keys included, a task's settings come from; it changes no row or config module, only a recipe's top-level key sets it, and Expand refuses it on a sub-recipe (D2).

Rationale: `arras` is benched against tapestry on the same inputs, so it must run with tapestry's settings.
Without the key, an `arras` task naming no profile would run silently on the bare config with no default profile, and one naming a profile would be refused for lacking an `arras` block, until an operator edits the file by hand.

### D9 — cartoon, a recipe holding tapestry's Discussion rows

`contracts/recipes/cartoon-recipe.yaml` (new) holds tapestry's Discussion-Write row and its Discussion-Review ring, with the same names, engines, whole config, segment and edges, bar the Discussion-Bouncer's `on_done`, which is empty so the block rewrites it to its exit.
It sets `version`, `entry` and `terminals` only at top level, and is embedded beside the other recipes.
It is a block source only: it is not in `shedrun.RecipeNames`, so no board entry can seed it as a run.
A test pins its rows equal to tapestry's Discussion rows, config included, so the two copies cannot drift while tapestry keeps its own.

Rationale: tapestry stays unchanged while `arras` is benched, so the Discussion rows live in two files until tapestry moves to the block; the pin makes the copy safe.

### D10 — arras, a seedable recipe beside tapestry

`contracts/recipes/arras-recipe.yaml` (new) is a loom recipe: Preflight, a block row `Cartoon` running `cartoon` with `block_gate: always` and `on_done: Drift-Baseline`, then tapestry's rows from Drift-Baseline to Friction-Reflect.
Its top-level keys equal tapestry's, with `profile_recipe: tapestry` added.
`arras` joins `shedrun.RecipeNames`, so a board task's `recipe` field may name it, and every site that accepts or wires tapestry by its name accepts and wires `arras` the same way, batten's Seed-Child and Run-Shed and the run reports included.
A test pins `arras`'s expanded rows equal to tapestry's rows, config included, plus the `Cartoon` row, bar Preflight's `on_done`, which is `Cartoon` in place of Discussion-Write, and `Cartoon`'s own edges; both recipes pass the loom contract check.
The board's `default_recipe` stays as it is; `arras` runs only where an entry names it.

Rationale: the block is proven on a real recipe whose behaviour must match tapestry's, which the pin states mechanically before any bench.

### D11 — Loom recipes expand in one place

`internal/loomrecipe` resolves every embedded recipe by name through one table, `cartoon` and `arras` included, and that table is the resolver it tells `shedbuild.Expand` (new).
Every loom accessor that parses an embedded recipe, the build, the routing projection, the row list and the row and engine lookups, reads the expanded recipe, so a consumer keyed on rows or engines sees `arras` as it sees tapestry.
`shedbuild` gains a routing projection from an already-expanded recipe, beside `shedbuild.RoutingOf`; it and `loomrecipe`'s row list carry each row's block path and skip target.
The batten and pawl recipes name no block and their loaders tell no resolver.

Rationale: one expansion point keeps the build, the status projection and the accessors from disagreeing on a recipe's rows.

### Added after Discussion (parent, 2026-11-03): Recipe names: motif and arras (operator, 2026-11-03)

Decision: The operator decided the names on 2026-11-03. The Discussion-only recipe, holding tapestry's Discussion rows and run as a block, is named motif, not cartoon: the motif is the design decided before the weaving. The new recipe beside tapestry that runs motif through a block and then tapestry's Plan and build rows is named arras. The name cartoon is kept free for a later Plan block, since a tapestry cartoon is the drawing the weaver follows, which here is the plan. Every place the record names cartoon reads motif.

Rationale: The operator approved arras and asked for weaving names on recipes, and judged cartoon to describe the plan rather than the Discussion.

## Constraints

- `PATTERN-sole-parsers`: the recipe is parsed only by `shedbuild.Parse`; `shedbuild.Expand` (new) works on its output and parses a sub-recipe through it.
- `PATTERN-shed-recipe-registry`: the `Block` producer is an entry of the one `Constructor` map, and the block gate checks join the same package as a second closed table (PATTERN changes).
- `PATTERN-loom-contract`: `arras` declares `contract: loom` and passes the contract check on its expanded rows; verbs decide by rows, so `arras` needs no verb change beyond accepting its name where tapestry's is accepted.
- `PATTERN-no-hard-coded-choice`: the sub-recipe, its gate check and the profile block are read from the recipe file and a table keyed by name.
- `PATTERN-named-constants`: the recipe names `cartoon` and `arras`, the engine `Block` and the check name `always` are each one named constant.
- `PATTERN-tolerant-records`: the history entry's new field is additive and omitted when empty.
- `PATTERN-shed-driver` and `PATTERN-shed-shared-machinery`: the block lives in shed's own packages, never in a loom package or a driver of its own.
- `PATTERN-test-speed`: every new test runs at the lowest tier that fits; expansion, routing and status are pure and need no tmux or agent.
- Tapestry's row names are durable resume identities, which inlined rows keep.

## Auto-mode assumptions

- The name `arras` (D10).
- `cartoon` as a block source only (D9).
- No `on_stuck` on a block row (D1, D4).
- The block row as a step of its own, and inlined rows keeping their names (D2, D7).
- `profile_recipe` in place of `arras` profile blocks (D8).

## Open risks

- The name `arras` awaits the operator's approval, which the task's scope requires; a rename touches only D9's and D10's names and constants.
- Peer run `decision-record-on-board` adds `approve_seam: discussion` to tapestry's Discussion-Bouncer row; whichever task lands second carries the key into `cartoon`, which D9's config pin enforces.

## Acceptance criteria

- A recipe whose block row sets an engine key, or lacks `block` or `block_gate`, and a hand-written row with engine `Block`, are refused at parse naming the row (D1).
- Expansion of a parent with one block yields the block row with engine `Block`, its `on_done` the sub-recipe's entry and its skip target the block's `on_done`, then the sub-recipe's rows under their own names with the block's path and every empty `on_done` among them rewritten to the block's exit; a parent holding a block whose recipe holds a block expands with each inner row's path `outer/inner` and each empty `on_done` set to its innermost block's exit (D2, D6).
- Expansion refuses an unknown block recipe, a nil resolver meeting a block row, a self-including block, an inlined row name or segment also used outside its block, an inlined `on_done`, `on_stuck` or `on_reroute` leaving its recipe as its recipe file sets it, a block row outside every block with an empty `on_done`, a block row whose `on_done` names itself or a row its expansion inlines, and a sub-recipe setting `contract`, `verify_source`, `amendment_target` or `profile_recipe` (D2).
- A run entering a block with a gate that holds records the block row Done, which marks entering the block and which `lym loom status` shows as entered, then runs the inlined rows; a gate that does not hold records the block row skipped with its reason and goes to the skip target with no inlined row recorded (D3).
- A goto onto the block row enters the block without evaluating its gate, and a re-launch onto the block row evaluates its gate (D3).
- The block ends at its exit when an inlined row whose expanded `on_done` is the exit returns Done or is skipped by its entry gate, or an inlined block row whose skip target is the exit is passed over, with no history entry of its own (D4).
- A Stuck with no `on_stuck` at an inlined row halts the run blocked at that row, and a resume calls that row again (D4).
- A `block_gate` naming no check is refused at build naming the row and the shipped checks, and `always` enters (D5).
- Every history entry of an inlined row, a goto entry included, carries the block path, and `lym loom status` shows it beside the row's name; a status file without the field still reads (D6, D11).
- On `arras`, the progress count numbers Preflight, Cartoon, Discussion-Write, Discussion-Review and then tapestry's later steps, and the Discussion-Review segment's budget comes from the discussion perch keys (D7).
- An `arras` task with no profile named takes tapestry's default profile, and a named profile's tapestry block, with a `profiles.yaml` that has no `arras` block (D8).
- `cartoon`'s rows equal tapestry's Discussion rows, config included, bar the Discussion-Bouncer's empty `on_done`, and `cartoon` is not a seedable recipe name (D9).
- `arras`'s expanded rows equal tapestry's rows plus the `Cartoon` row, config included, bar Preflight's `on_done` and `Cartoon`'s own edges; both pass the loom contract check; a board task naming `arras` is seeded and run by batten as a tapestry task is; this task makes no change to tapestry's recipe file (D10).
- The loom build, the routing projection and the row and engine accessors return the same expanded rows for `arras`, block path and skip target included (D11).

## PATTERN changes

- `PATTERN-shed-recipe-registry` — changes: Adding producer, recipe row or block gate: one `Constructor` or `BlockGates` map in `internal/shedrecipe`, bar `Implementer`'s; a row's value looked up by key, a side action a `Command` row, a row off by one config key its `entry_gate`, one whose off reads more its own call.

## Notes for the plan writer

- The sites that name tapestry by name, rather than by rows, are where `arras` must be accepted too: `internal/battencli/wire.go`'s supported child recipes, `internal/reportengine/runs.go`, `internal/reportengine/timingspans.go` (whose phase table also needs a phase for `Cartoon`), `internal/loomrecipe/recipekeys.go`'s retired rows and the profile lookups Seed-Child makes in `internal/battencli/runsettings.go`; grep for `RecipeTapestry` for the rest.
- `internal/loomrecipe/names.go`'s `RecipeEngines` and the bouncer and finalize row helpers parse each embedded source directly; route them through the expanded parse.
- The skip in `internal/shedengine/entry.go` routes, persists and finds the next row by `OnDone` in several places, and the goto admission in `internal/shedengine/goto.go` reads a skipped entry's routed row from `OnDone`; each takes the skip target when set.
