# Split tapestry into Motif and Jacquard blocks, with a Record-Seed row

A task whose decision record was approved earlier could only be built by a tapestry run, which gates its Discussion through the Motif block.
This change gives that build its own recipe, `jacquard`, and makes tapestry the composition of the two: the Motif block followed by the Jacquard block, each row defined once.

## What changes

- **`contracts/recipes/jacquard-recipe.yaml`** (new) holds Preflight, a new Record-Seed row, and every row tapestry held from Drift-Baseline to Friction-Reflect.
  The file is tapestry's old file moved, so those rows keep their routing, segments, gates and comments.
  Its top-level keys are `entry: Preflight` and tapestry's `amendment_target`, `contract`, `verify_source` and `terminals`.
- **`contracts/recipes/tapestry-recipe.yaml`** is two block rows, entry `Motif`:
  `Motif` (`block_gate: undiscussed`, `on_done: Jacquard`) and `Jacquard` (`block_gate: always`, no `on_done`).
  Preflight now runs after Discussion in a tapestry run.
  Row names are unchanged, so a tapestry run started before this change resumes under the new graph by row name.
- **`shedbuild.Expand`** admits a block row with an empty exit when it is the last row of its recipe and its gate is `always`.
  The rows it inlines keep an empty `on_done`, so Friction-Reflect still ends the run.
  A last such row with any other gate is refused with the way forward "set on_done, or use block_gate always", since its skip would fall back to the block's own entry.
  Every other block row with an empty exit is refused as before.
- **The loom contract** (`internal/shedbuild/loomcontract.go`) now takes at most one record row (DiscussionWrite or RecordWrite) and at most one seed row (`RecordSeed`), and at least one of the two.
  With both, the seed row must follow the record row on the on_done path from the entry.
  Every later clause reads the seed row as the record row when there is no record row.
- **`RecordSeed`** is a new registry engine (`shedrecipe.LoomSeedEngine`), whose producer `loomshed.NewRecordSeed` is Record-Write in a stored-only mode.
  It writes whichever of the decision record and support log is missing, the record from the entry's stored record, and commits the pair.
  An existing record is never overwritten, so after tapestry's Discussion the row writes nothing.
  With the record missing and no stored record, the row is Stuck with nothing written.
  Its way forward depends on the recipe:
  - a recipe with a Discussion row goes back to that row with `lym loom goto`;
  - a recipe without one is re-run as tapestry.
    The reason names one sequence for a pair batten made and one for any other pair, filled in with the slug and with the seed's path, driver and parent.
- **Vocabulary and lookups:** `shedrun.RecipeJacquard` joins the closed recipe vocabulary, `shedcli` arms it, and `recipes.go` embeds it.
  `loomrecipe` resolves it and lists it beside tapestry in `RecipeEngines`, `BouncerRunSubdir` and `IsFinalizeRow`.
  `loomrecipe.RecordEngine` returns `RecordSeed` for a recipe whose only record source is a seed row, and still returns `DiscussionWrite` for tapestry.
- **Wiring:** `internal/loomcli/wiring.go` fills `Env.RecordSeed` from Record-Write's paths, stored-record read and commit.
  It adds the recipe's Discussion row, read through `loomrecipe.EngineRow`, the slug, the seed path and a seed reader.
  `testkit/envkit` and shedrecipe's test Env fill it too.
  `loomshed.NameRecordSeed` and `loomshed.NameJacquard` are the new durable row names, with interrupt policy reinvoke.

## Design decisions

- Record-Seed picks its way forward from the recipe's rows, never the recipe name:
  a non-empty Discussion row means "go back to Discussion".
- `Expand`'s run-ending rule reads the block row's resolved exit, not only its own `on_done`.
  So a block nested in a run-ending block is held to the same rule: it must be last and always gated.
- The decision verbs' RecordWrite-path refusals in `internal/loomcli/decision.go` still name Record-Write.
  The record limits this change to Record-Seed's engine and contract steps, and leaves those refusals, batten's lists and the report's phase tables to later changes.
  Until then a tapestry timing report counts Record-Seed's and Jacquard's time under "other".
- When the record exists but the support log is missing and the entry has no stored record, Record-Seed writes the board-rendered support log, as Record-Write does.

## Tests

- `internal/shedbuild`: Expand admits a last always-gated block row with no `on_done` and keeps its inlined last row's empty `on_done`.
  It refuses such a row whose gate is not `always`, and still refuses one that is not last.
  The loom contract refuses two record rows, two seed rows, neither, and a seed row ahead of its record row.
  It admits a seed row beside or in place of the record row.
- `internal/loomshed`: Record-Seed seeds a missing record from the stored record and leaves an existing one byte for byte.
  With no stored record it is Stuck with nothing written, its reason naming the Discussion goto or the batten and re-seed sequences.
- `internal/shedrecipe`: the RecordSeed entry refuses a relative path, an empty slug and each nil seam, and builds over a filled Env.
- `internal/loomrecipe`:
  - the expanded tapestry rows equal the pre-block rows with only the edges the blocks add, and jacquard's top-level keys other than `entry` equal tapestry's;
  - jacquard builds and runs alone from Preflight to Friction-Reflect;
  - the recipe walk of an undiscussed entry runs Motif, the Discussion rows, Jacquard, Preflight and Record-Seed, which writes nothing, then Drift-Baseline;
  - a discussed entry's run passes over Motif and Record-Seed writes nothing;
  - the walks and resume tests fake Preflight by name, since Motif is now the first row.

## Merge danger

**Door:** two-way
**Blast radius:** loom
