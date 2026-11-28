## Goal

Add `jacquard`, a loom recipe that holds tapestry's rows after Discussion.
It runs only on a board entry that holds a stored decision record, and refuses any other at start.
Tapestry stays a recipe and is made of the motif block followed by the jacquard block, so each part has one source.
Raddle names jacquard for the parts it files from a Sketch.
The glossary gains `jacquard`, and the repository's nicknames gain "jacq".

## Scope

In: the jacquard recipe and its registration, tapestry's rewrite over it, the block, engine and loom-contract changes that rewrite needs, the start refusal, raddle, run profiles, recipe lists, cost cells, the glossary and the docs naming recipes.

Out:

- Plan-ahead chains (`pipelined-chain-planning`), which builds on this recipe.
- A move of the `tapestry.*` config keys.
- A rename of raddle's `tapestry` proposal shape.
- The board's default recipe, which stays tapestry.

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
It keeps its own top-level keys, since an expansion reads none of a block's, and they equal jacquard's.
Preflight now runs after Discussion in a tapestry run.

Row names are unchanged, and a tapestry run started before this change resumes under the new graph by row name.
A run parked inside the Motif block goes on to Preflight and Record-Seed, which finds the approved record and writes nothing.
A run parked at Preflight that never entered Motif goes on to Record-Seed:
with a stored record it seeds it, which is what the Motif gate would have done, and with none Record-Seed stops it per D4.
The rows from Preflight on carry block path `Jacquard`, so the run checklist shows them, and the Plan-Review and Code-Review segments, under the Jacquard block.
History entries an in-flight run wrote before the change carry no block; the checklist joins them to their rows by name, so only their block label differs.

Rationale: tapestry's rows are motif's plus jacquard's with no copy.
Preflight's checks guard config the build rows read, which Discussion does not, and a Preflight halt after Discussion keeps the approved record.

### D3 — A last outermost block row may end the run

`shedbuild.Expand` admits an outermost block row with an empty `on_done` when it is the last row of its recipe.
The rows it inlines keep an empty `on_done` where they set none, so the row that ends the sub-recipe run alone ends the parent run.
Every other outermost block row with an empty `on_done` is refused as today.
A nested block row's empty `on_done` still takes the exit of the block enclosing it.

Bound: the edge lets an empty `on_done` end the run only inside a block that is the parent's last row.
For a loom recipe the loom contract's terminal check still requires Finalize to be followed only by terminals.

Rationale: tapestry's Jacquard block has no row after it, and Friction-Reflect's empty `on_done` is what ends a run quietly.

### D4 — A Record-Seed engine seeds the stored record, and the loom contract admits it

A new registry engine, `RecordSeed`, runs as row `Record-Seed`.
It is Record-Write's producer in a stored-only mode: it writes whichever of the decision record and support log is missing, the record from the entry's stored record, and commits the pair.
A record that exists is never overwritten, so an approved record and every entry added after Discussion survive.
With the record missing and no stored record, the row is Stuck with nothing written.
Its reason names a way forward per recipe:

- a jacquard run: `lym batten cancel <slug> --remove` from the hub's prime worktree, `lym board upsert '{"slug":"<slug>","recipe":"tapestry"}'`, `lym board set-status '{"slug":"<slug>","status":null}'`, then `lym batten run <slug>`;
- a tapestry run: `lym loom goto --to Discussion-Write`, then `lym loom resume`.

The loom contract in `internal/shedbuild/loomcontract.go` changes from "one record row" to:
a loom recipe holds at most one record row (DiscussionWrite or RecordWrite) and at most one seed row (RecordSeed), and at least one of the two.
When it holds both, the seed row follows the record row on the on_done path from the entry.
Every other clause reads the seed row as the record row when the recipe holds no record row.
`loomrecipe.RecordEngine` keeps returning the record row's engine, so tapestry's decision verbs read Discussion as today, and returns RecordSeed for a recipe with only a seed row.
The decision verbs treat RecordSeed as they treat RecordWrite, with Record-Seed among the rows before the record is written.
Darn's Record-Write row is unchanged.

Bound: a seed row writes only from the stored record, never from the entry body, and the contract admits it only beside or in place of the one record row.

Rationale: jacquard needs the seed tapestry's Motif gate gives a discussed entry, and the contract needs one row to name the record's source.
Writing only a missing file makes the row a no-op after tapestry's Discussion, so the seed never clobbers the run's record.

### D5 — Jacquard's start refusal is judged by the recipe's rows

One predicate in `internal/loomrecipe` reports whether a recipe needs a stored record: its expanded rows hold a seed row and no record row.
It is true for jacquard and false for tapestry, darn, motif and raddle.
A needing recipe refuses the start of an entry that is not flagged discussed with its record file on the board, the same reading raddle's start guard makes.
The refusal is one message, `<slug> runs the jacquard recipe, which builds from a stored decision record, and it holds none`, with the way forward `lym board upsert '{"slug":"<slug>","recipe":"tapestry"}'`, which a start with no status file can take.
It is judged in two places:

- batten's start guard, for a run or step with no status file, beside raddle's check, so no worktree is made;
- `lym loom start`, before it writes the seed, for a start made in a worktree directly.

A resume passes over it, and Record-Seed's Stuck guards a run whose entry lost its record before the row ran.

Bound: the check reads the board once at start and nothing else; it refuses only a needing recipe on an entry with no stored record.

Rationale: `PATTERN-shed-recipe-registry` has a loom verb refuse by a recipe's rows, never its name.

### D6 — Raddle names jacquard for the parts it files

`internal/raddleparser`'s part recipe vocabulary becomes jacquard or darn.
A part naming tapestry is a `recipe-invalid` finding whose detail names jacquard as the recipe for a part filed from a Sketch.
The raddle stencil, `contracts/specs/raddle-proposal-spec.md`, the raddle review rubric and the loom raddle context's profile listing say jacquard where they name the part recipe tapestry.
Filing copies the part's recipe, so a filed part carries jacquard with its stored record and the discussed flag, which filing sets today.

Bound: filing re-runs the check, so a proposal approved before this change with a tapestry part is sent back at filing, rewritten to jacquard and approved again; nothing is filed under tapestry.

Rationale: a part filed from a Sketch always has its record, so naming tapestry leaves whether Discussion runs to a flag.

### D7 — Every run profile gains a jacquard block

Each profile of `internal/runprofile/template.yaml` that holds a tapestry block gains a `jacquard` block with the same keys minus `loom.discussion`.
`defaults` gains `jacquard: solid`, as tapestry's.
An existing hub's profiles file gains the blocks at its next reconcile, since a key absent from both the file and its `profiles.template-base` takes the template's value.

Rationale: a run reads its recipe's block, and raddle's check refuses a part whose profile holds no block for its recipe.

### D8 — Closed recipe lists admit jacquard where it walks tapestry's path

Every closed recipe list in Go that names tapestry for a path a jacquard run also walks names jacquard too:
batten's admitted child recipes and their refusal text, `internal/loomrecipe`'s recipe lists, and the report's selected run recipes.
The report's phase table for jacquard is tapestry's without its Discussion families.
Both tables put the Jacquard block row and Record-Seed in Preflight's phase.
A default that falls back to tapestry when a run has no seed stays tapestry.

Rationale: a jacquard run builds, lands and reports as a tapestry run of a discussed entry does.

### D9 — Cost cells count a discussed tapestry run as jacquard

`internal/reportengine`'s cost cells key a tapestry run whose `stored_record` is true under jacquard.
Run reports and phase tables keep the recipe the run's seed names.

Bound: only the cost-cell key changes, and only for tapestry runs with `stored_record` true; the tapestry cell with `stored_record` true is left empty.

Rationale: those runs walked jacquard's rows, and raddle's cost lines for jacquard parts would otherwise read too few runs for weeks.

### D10 — The glossary and docs name jacquard

`internal/heddleengine/glossary.md` gains the line:
`- **jacquard**: the recipe that builds a task from its stored decision record: tapestry's rows after Discussion.`
`docs/reference/weaving-glossary.md` gains a jacquard row, and its tapestry row says tapestry is motif followed by jacquard.
`docs/overview.md`'s recipe mentions and the tapestry and motif recipe headers name jacquard where they list recipes or describe tapestry's rows.

Rationale: the operator asked for the glossary term, and the docs list each recipe.

### D11 — The "jacq" nickname is written through heddle after landing

The nickname line `- Read "jacq" as the jacquard recipe: tapestry's rows after Discussion, run on an entry with a stored decision record.` joins the `nicknames` entry of Bolt's `heddle.json`.
The run's parent writes it with `lym heddle set` once the run has landed; no card of this run writes it.

Rationale: `PATTERN-heddle-instructions` has `heddle.json` written only through `lym heddle`, and it is the repository layer in Bolt, not a file of this worktree.

## Constraints

- `PATTERN-shed-recipe-registry`: jacquard declares `contract: loom` in its standard form; the start refusal reads rows, not the name; RecordSeed is one row of the registry map.
- `PATTERN-code-conventions`: the recipe name is one named constant in `shedrun`; the refusal fills it from that constant.
- `PATTERN-refusal-way-forward`: the start refusal and Record-Seed's Stuck name their ways forward, and each lands with a test that reaches it.
- `PATTERN-config-layers`: no config key moves; the new profile blocks are template additions the hub reconcile carries.
- `PATTERN-heddle-instructions`: the nickname changes only through `lym heddle set`.
- Fact, from `internal/shedbuild/expand.go`: a block's rows keep their names, segments must not clash with rows outside, and an outermost block row with an empty `on_done` is refused today.
- Fact, from `internal/shedbuild/loomcontract.go`: the loom contract today refuses a recipe with other than one DiscussionWrite or RecordWrite row.
- Fact, from `internal/loomshed/recordwrite.go`: Record-Write writes only a missing record or support log and commits every call.
- Fact, from `internal/boardengine/raddlefile.go` and `internal/loomshed/raddlefile.go`: filing writes each part's record, sets its discussed flag and re-runs the proposal check first.
- Fact, from `internal/runprofile/doc.go`: the reconcile gives a profiles file each template key it holds as its base does, gained ones included.

## Auto-mode assumptions

- Config keys stay under `tapestry.*` (D1).
- Preflight runs after Discussion in tapestry (D2).
- The seed is its own engine under a widened loom contract (D4).
- Raddle's `tapestry` shape keeps its name (Scope).
- A jacquard profile block is tapestry's minus `loom.discussion` (D7).
- The nickname is written by the parent after landing (D11).

## Open risks

- D11 asks the run's parent to run `lym heddle set` for the `nicknames` entry once the run lands; the entry asks the task to add it, and no card of the run can.

## Acceptance criteria

- D1, D2: the expanded tapestry recipe's rows equal the Motif block's rows followed by the Jacquard block's rows, in order with the same edges, and jacquard's top-level keys equal tapestry's (tier 0, `internal/loomrecipe`).
- D1: `jacquard` is in the recipe-name vocabulary and builds as a loom recipe with Preflight as its entry and Friction-Reflect as its terminal (tier 0).
- D2: a tapestry run of an undiscussed entry walks Motif, Discussion, Preflight, Record-Seed with nothing written, then Drift-Baseline; one of a discussed entry passes over Motif with the record seeded and Record-Seed writes nothing (tier 0, a recipe walk over the real rows).
- D3: Expand admits a last outermost block row with no `on_done`, whose inlined terminal row keeps an empty `on_done`; it still refuses an outermost block row with no `on_done` that is not last (tier 0, `internal/shedbuild`).
- D4: Record-Seed seeds a missing record from the stored record, leaves an existing record untouched, and with no stored record is Stuck with nothing written and a reason naming both ways forward (tier 0, `internal/loomshed`).
- D4: the loom contract builds tapestry, jacquard, darn and motif, and refuses a recipe with two record rows, two seed rows, neither, or a seed row ahead of its record row (tier 0, `internal/shedbuild`).
- D4: `loomrecipe.RecordEngine` returns DiscussionWrite for tapestry and RecordSeed for jacquard, and `lym loom decision add` on a jacquard run before and after Record-Seed takes the RecordWrite path (tier 0 and tier 1).
- D4: a tapestry run Stuck at Record-Seed moves with `lym loom goto --to Discussion-Write` and runs Discussion on resume (tier 1).
- D5: `lym batten run` of a jacquard entry with no stored record is refused before a worktree is made, and `lym loom start` refuses the same; both messages carry the way forward; an entry with a stored record starts, and an undiscussed tapestry entry starts (tier 1).
- D5: the needs-stored-record predicate is true for jacquard and false for tapestry, darn, motif and raddle (tier 0).
- D6: raddle's check refuses a part naming tapestry with a detail naming jacquard and accepts jacquard and darn; a filed jacquard part carries recipe jacquard, the discussed flag and its record (tier 0).
- D7: every profile holding a tapestry block holds a jacquard block without `loom.discussion`, and the defaults name jacquard (tier 0, `internal/runprofile`).
- D8: batten admits jacquard as a child recipe, and the report selects a jacquard run with its phase table (tier 0).
- D9: a tapestry run with `stored_record` true lands in the jacquard cost cell, and one with false stays in tapestry's (tier 0, `internal/reportengine`).
- Review: D6 — the raddle stencil, `contracts/specs/raddle-proposal-spec.md` and the raddle review rubric name jacquard as the part recipe.
- Review: D10 — `internal/heddleengine/glossary.md` carries the jacquard line, and `docs/reference/weaving-glossary.md` and `docs/overview.md` name jacquard and tapestry's two blocks.
- Review: D11 — the change description asks the parent to run `lym heddle set` for the nickname after landing.

## Notes for the plan writer

- The recipe lists D8 covers are found by grepping for `shedrun.RecipeTapestry` outside tests.
