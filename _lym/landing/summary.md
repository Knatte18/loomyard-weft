# Add the jacquard recipe; tapestry becomes its Motif and Jacquard blocks

`contracts/recipes/jacquard-recipe.yaml` is a new recipe that plans, builds and lands a task from a decision record
approved before the run, with no Discussion.
It holds `Preflight`, a new `Record-Seed` row, and every row tapestry held from `Drift-Baseline` to `Friction-Reflect`.
Those rows moved with their routing and comments; they are not copied.
`tapestry-recipe.yaml` is now two block rows: `Motif` (`block_gate: undiscussed`, `on_done: Jacquard`)
and `Jacquard` (`block_gate: always`, no `on_done`), with `entry: Motif`.
The build rows are then defined once, for a discussed and an undiscussed run alike.
`jacquard` joins the recipe-name vocabulary, `lym shed`'s recipe table and the embedded recipes,
and its rows keep reading the `tapestry.*` and `loom.*` config keys.

## What changes in a tapestry run

- `Preflight` runs after Discussion instead of first.
  A Preflight halt there keeps the approved record,
  and a worktree or fabric fault in an undiscussed run shows at Discussion first.
- `Record-Seed` follows `Preflight`.
  After Discussion, or after `Motif` seeded a discussed entry, the record exists and the row does not rewrite it.
- The rows from `Preflight` on carry block path `Jacquard`.
- Row names are unchanged, so a run started under the old graph resumes by row name under the new one.

## Design decisions

- **`Record-Seed`** (engine `RecordSeed`) is `Record-Write`'s producer in a stored-only mode.
  It writes only a missing decision record or support log, never from the entry's body, and commits the pair.
  With no record and no stored record it is Stuck with nothing written.
  The reason names `lym loom goto --to Discussion-Write` for a run whose recipe holds Discussion,
  and otherwise the steps that re-run the task as `tapestry`,
  both for a pair batten made and for any other pair.
- **A last block row may end the run.**
  `shedbuild.Expand` admits an outermost block row with no `on_done` only when it is the recipe's last row
  and its `block_gate` is `always`, so no skip can fall back to the block's own entry.
- **The loom contract** now takes at most one record row and at most one seed row, and at least one of the two.
  With both, the seed row follows the record row on the `on_done` path.
  With no record row, the seed row stands for it.
- `loomrecipe.RecordEngine` still returns `DiscussionWrite` for tapestry, and returns `RecordSeed` for jacquard.
- The seed's parent parameter name is now the constant `loomshed.SeedParentParam`.

## Not covered for a standalone `jacquard` run

The recipe builds and is a legal seed value, but running it alone is not supported end to end:

- `lym batten run` refuses it as a child recipe.
- The run-profile template has no `jacquard` block.
- The `lym loom decision` verbs do not yet treat `RecordSeed` as they treat `RecordWrite`,
  so they handle a `jacquard` run as one that is not record-written.
- `lym loom start` gains no check of whether the task's entry is discussed.
- The timing report does not select `jacquard` runs.

## Tests

The tests cover that:

- the expanded tapestry rows are the `Motif` block's followed by the `Jacquard` block's,
  and jacquard's top-level keys other than `entry` equal tapestry's (`TestTapestryBlocks`);
- `jacquard` builds alone as a loom recipe from `Preflight` to `Friction-Reflect` (`TestJacquardRecipeRunsAlone`);
- a recipe walk of an undiscussed and of a discussed entry reaches `Record-Seed` with the record present,
  and the row does not read the stored record
  (`TestSequence_FullRunBlocksAtMergeIn`, `TestSequence_DiscussedEntryPassesOverMotif`);
- `Expand` admits a last always-gated block with no `on_done`,
  and refuses one that is not last or not always gated (`internal/shedbuild/expand_test.go`);
- `Record-Seed` seeds, keeps, or is Stuck with nothing written (`internal/loomshed/recordseed_test.go`);
- the loom contract refuses two record rows, two seed rows, neither,
  and a seed row ahead of its record row (`internal/shedbuild/build_test.go`).

## To check

- A tapestry run parked at `Preflight` that never entered `Motif` now goes on to `Record-Seed`, not to Discussion.
  With a stored record it is seeded from it; without one the run halts there with the way back to `Discussion-Write`.
  No in-flight run was resumed by hand against the new graph.
- No live `tapestry` or `jacquard` run was driven by hand.
- The timing report's tapestry phase table names neither `Record-Seed` nor `Jacquard`.

## Merge danger

**Door:** one-way
**Blast radius:** loom

A revert does not rewrite records that runs have already written:
a status can name `Record-Seed` or `Jacquard`, and a seed can record recipe `jacquard`, which the old binary lacks.
