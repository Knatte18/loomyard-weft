```yaml
lens: code-correctness
outcome: reported
```

Task: run the code-correctness lens over the diff, one case per function or branch.

**Per function or branch: case run, result**

- `Expand`, no block row: returned as is. Correct.
- `Expand`, top-level block row with empty `on_done`: refused naming the row and a way forward. Correct.
- `Expand`, a block inside a block: the inner block row's empty `on_done` takes the outer block's exit, which also becomes its skip target. The inner rows' empty `on_done` take the inner exit, and their paths read `outer/inner`. Correct.
- `Expand`, a block that includes itself or loops back to a recipe already entered: `chain` catches any cycle that doesn't start at the top recipe. A cycle back to the top recipe is not caught by `chain`, but the row-name collision check refuses it first, so there is no infinite recursion. The message names a row collision, not a cycle, which is acceptable.
- `Expand`, nil resolver with a block row: refused. Correct. Unknown recipe and unparsable bytes are also refused, with the cause wrapped.
- `Expand`, a sub-recipe setting `contract`, `verify_source`, `amendment_target` or `profile_recipe`: refused. Correct.
- `Expand`, an edge naming a row outside the sub-recipe: refused. A rewritten empty `on_done` is exempt, because the check runs on the sub-recipe's parsed rows before the rewrite. Correct.
- `Expand`, name and segment collisions: names and segments are registered up front for the top level and after each inline. Sibling rows of one block sharing a segment are allowed. Correct.
- `Expand`, `on_done` naming the block row or an inlined row (nested block rows included): refused. The check runs after the rows are expanded. Correct.
- `Parse`, `checkRowShape` and `checkBlockRowShape`:
  - A block row setting any of `engine`, `config`, `on_stuck`, `on_reroute`, `segment`, `max_bounces` or `entry_gate` is refused, with all the keys listed.
  - A missing `block` or `block_gate` is refused.
  - A hand-written `engine: Block` without block keys is refused.
  - An empty `on_done` on a block row is allowed.
  - Correct.
- `Build`, `refuseUnexpandedBlockRow` and `rowEntryGate`: an unexpanded block row is refused before the contract check. An unknown `block_gate` is refused naming the row, via `LookupBlockGate`, which lists the shipped checks. `always` holds with an empty reason. `Block` producer: `blockEntry` accepts a nil config and `Call` returns `Done`. Correct.
- `RoutingOf` and `RoutingFrom`: `RoutingOf` refuses an unexpanded block row, and `RoutingFrom` copies `SkipTo` and `Block`. Correct.
- `shedengine.enter`:
  - A gate that holds persists the row as entered.
  - A gate that does not hold appends a skipped entry carrying `Block` and routes to `skipTarget()`. `SkipTo` is empty on every existing row, so those still route by `OnDone`.
  - A skip to a row that is paused hands back `Next` as the skip target.
  - Correct.
- `validate`, `SkipTo` naming no producer: refused. Correct. A `SkipTo` equal to the row's own name is not checked, but `Expand` cannot produce one.
- `gotoAdmitted`: the routed row after a `Done` is `OnDone`, and after a skip it is `skipTarget()`. Correct.
- `Goto` and `Relaunch` history: both carry `target.Block`. In `Relaunch` the lookup runs after `CheckGotoTarget` has admitted the target, so it is found. `Goto` rebinds `target, ok :=` with the old `ok` use removed. Correct.
- History writers: `Block: def.Block` is set in `run.go:352`, the three in `reroute.go`, `enter.go:51` and both goto sites. I saw no `HistoryEntry` constructor in the engine that lacks it. `omitempty` keeps older status files readable. Correct.
- `Routing.mainLine` and `ProgressAt`: the main line follows `OnDone`, so it runs block row, sub-recipe entry, inlined rows, then the exit, and the block row counts as a step. After a pass-over, `current` is the exit, so the step number jumps without any change to `progress.go`. Correct.
- `loomrecipe.parseEmbeddedRecipe`, `Rows`, `Routing`, `reviewSegments`, `RecipeEngines` and the bouncer and finalize helpers: all go through the expansion, and the resolver is `embeddedRecipeSource`. Correct. Not counted: `names.go` `RecipeEngines` does not include motif, which is expected.
- `loomrecipe.ProfileRecipe`: an empty name gives `tapestry`, `arras` gives `tapestry`, and a recipe with no key gives its own name. Correct.
- `battencli.prepareRunSettings`: an error from `ProfileRecipe` is swallowed, and the task's own recipe name is used so a missing profile refuses as before. This is documented and intentional.
- `indefiniteArticle` in `arbiterprompt.go`: it indexes `label[0]`, and `recipeLabel` never returns an empty string, so it is safe. `arras` gives "an arras run". Correct.
- `reportengine.arrasPhaseTable`: `tapestryPhaseTable()` allocates a fresh map on every call, so adding the Motif family does not mutate a shared table. Correct.
- `shedverbs.blockAwareLast` and `RenderStatusHuman`: see F1.
- `loomcli` `checklistVisitsOf`: a `Done` on a block row reads `entered`, and the visit carries `entry.Block`. Correct.

**Findings**

- **F1, LOW, design.** `internal/shedverbs/human.go` (`blockAwareLast`, the `strings.HasPrefix(last, latest.Producer)` test).
  - The block name is spliced in only when the activity's `last` line starts with the producer name.
  - A row whose name is a prefix of another row's, such as `Discussion-Write` and a hypothetical `Discussion-Write-2`, makes the test fire on the wrong text. Nothing in a produced `last` line pins this, since it is free text.
  - Separately, for a block row's `Done` the whole `last` line is replaced by `&lt;row&gt; → entered`, which drops any routed-to detail the activity carried.
  - Evidence: `human.go` lines 867-884 of the diff.
  - Suggested fix: match on the producer followed by a space or end of line.
- **F2, LOW, design.** `internal/shedverbs/human.go` (`blockAwareLast`, the `def.SkipTo != ""` test).
  - It identifies a block row by a non-empty skip target.
  - `ProducerDef.SkipTo` is documented as available to any row ("such as the row that opens a block"), so a non-block row that later sets `SkipTo` would also read "entered".
  - Today only `Expand` sets it, so there is no wrong result now.
  - Suggested fix: identify the block row some other way, or document that `SkipTo` marks a block row.

I found no `design` defect at `BLOCKING` or `MEDIUM` severity in the engine, `Expand`, `Parse`, `Build` or the loom routing paths.

**Tool calls**
1. Read `internal/shedengine/progress.go`: the diff showed only its comments, and I needed the `mainLine` and `ProgressAt` logic to check the jump on a pass-over.
2. `grep "HistoryEntry{" internal/shedengine`: to confirm every history writer carries `Block`.
