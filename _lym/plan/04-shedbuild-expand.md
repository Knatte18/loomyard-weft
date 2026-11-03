# Card 4 — shedbuild-expand

**Create:**
- `plan:internal/shedbuild#Expand` -> `func Expand(r Recipe, resolve func(name string) ([]byte, error)) (Recipe, error)`
- `plan:internal/shedbuild#TestExpand` -> `func TestExpand(t *testing.T)`

**Extends:**
- `internal/shedbuild/doc.go`

**Uses:**
- `plan:internal/shedrecipe#EngineBlock`

**Intent:** Inline each block's sub-recipe into its parent at build time (D2), in a new file `internal/shedbuild/expand.go`, beside `Parse`.
`Expand` takes a parsed recipe and a told resolver from a recipe name to its bytes, parses each sub-recipe through `Parse` and returns a new `Recipe`; it never mutates its argument.
A recipe with no block row is returned as it is, whatever the resolver.

For each block row, in order:
- the sub-recipe is resolved, parsed and expanded first, its own blocks included (innermost first);
- the block row is emitted with `Engine` set to `shedrecipe.EngineBlock`, `SkipTo` set to the block's exit and `OnDone` set to the sub-recipe's `entry`, `Block` and `BlockGate` kept;
- the sub-recipe's rows follow it, under their own names.
A block's exit is its row's `on_done` as its recipe file sets it, or, when that is empty, the exit of the block enclosing it.
Every inlined row's empty `on_done`, and an inlined block row's empty skip target, become the exit of the innermost block enclosing that row.
Each inlined row's `BlockPath` is the names of the block rows enclosing it, outermost first, joined by `/`; the outermost block row and every row outside a block carry none.
A sub-recipe's `terminals` are not read; the parent's stay the run's.

Refusals, each prefixed `shedbuild: block %q:` with the block row's name, then the case and its way forward:
- the resolver refuses the block's recipe: `recipe %q: <resolver error>; way forward: set block to an embedded recipe's name`;
- a nil resolver meets a block row: `no recipe resolver is told; way forward: expand the recipe through a caller that tells the resolver of its embedded recipes`;
- the block's recipe names itself, directly or through its own blocks: `recipe %q includes itself through <chain joined by " -> ">; way forward: remove the block that closes the cycle`;
- an inlined row's name equals a row outside its block: `row %q of recipe %q is also a row outside the block; way forward: rename one of the two rows, or use the recipe in one block only`;
- an inlined row's non-empty `segment` is used by a row outside its block: `segment %q of row %q is also used outside the block; way forward: give the block's rows a segment of their own`;
- an inlined row's `on_done`, `on_stuck` or `on_reroute`, as its recipe file sets it, is non-empty and names no row of that recipe file: `row %q of recipe %q sets %s %q, which names no row of that recipe; way forward: route the row inside its recipe, or leave on_done empty to reach the block's exit`; the edges expansion sets are exempt;
- a block row outside every block has an empty `on_done`: `on_done is empty; way forward: set on_done to the row the run takes after the block`;
- a block row's `on_done` names the block row itself or a row its own expansion inlines: `on_done %q lies inside the block; way forward: set on_done to a row outside the block`;
- a sub-recipe sets `contract`, `verify_source`, `amendment_target` or `profile_recipe`: `recipe %q sets %s, which belongs to the run; way forward: remove %s from the block's recipe`.
Bound: the same recipe in two blocks of one parent is refused by the row-name check.

`doc.go` gains a `# Blocks` section stating the above.

Tests, tier0: `TestExpand` (new), a table over a map resolver of YAML sources built with `Stub` rows:
- a parent with one block yields the block row with engine `Block`, `OnDone` the sub-recipe's entry and `SkipTo` the block's `on_done`, then the sub-recipe's rows under their own names with `BlockPath` the block's name and every empty `on_done` rewritten to the exit;
- a parent whose block's recipe holds a block yields each inner row's path `Outer/Inner`, each empty `on_done` set to its innermost block's exit, and an inner block row with an empty `on_done` taking the outer exit as its skip target;
- a sub-recipe whose `terminals` names a row that is not its own expands, and the result keeps the parent's `terminals`;
- a recipe with no block is returned unchanged with a nil resolver;
- one refusal row per case, each asserting the block row's name and the way forward's text: the resolver refusing, a nil resolver, a block naming itself directly, a block reaching itself through an inner block, a shared row name, a shared segment, an inlined `on_done`, `on_stuck` and `on_reroute` each naming no row of its recipe (three rows), an outermost block row with an empty `on_done`, a block row's `on_done` naming itself and naming an inlined row (two rows), and a sub-recipe setting each of `contract`, `verify_source`, `amendment_target` and `profile_recipe` (four rows).

**ImpactSummary:** New pure function in `internal/shedbuild`; no caller until card 6, and `Parse` and `Build` are unchanged.
