# Card 20 — peers-per-recipe

**Create:**
- `internal/loomcli#peerRecipe`

**Edit:**
- `internal/loomcli#peersDeps`
- `internal/loomcli#peersVerb`
- `internal/loomcli#hubPeersDeps`
- `internal/loomcli#peersFakeDeps`
- `internal/loomcli#TestPeersVerb`
- `internal/loomcli#TestHubPeersDeps_OverRealHub`

**Uses:**
- `internal/loomcli#recipeRow`
- `internal/loomcli/doc.go`

**Intent:** `lym loom peers` reads each peer's row position against that peer's own recipe (D3), where today it reads every peer against tapestry's rows.

- `peersDeps.loomRows` is replaced by `rows func(task boardengine.Task) []string`: the row names of the peer's recipe, in file order. `peersVerb` reads each listed peer's `Row`, and the caller's `own_row`, through `recipeRow(task.Status, deps.rows(task))`; `peerLine.Row`'s doc says it is the position in the peer's own recipe.
- `hubPeersDeps` fills `rows` from `peerRecipe`. When the peer's worktree is on this machine (`fabricengine.WorktreePath` resolves) and holds a self-run seed, the recipe is the seed's, an empty one reading as tapestry. Otherwise it is the recipe Seed-Child seeds for the board entry: `shedrun.RecipePawl` for a project entry, else the entry's `recipe` field, empty meaning tapestry, the rule `boardRecipe` in `internal/battencli/wire.go` applies, which `peerRecipe`'s doc names as its twin since a cli package never imports another.
- `loomrecipe.RowNames` of that recipe gives the rows; a recipe that names no embedded recipe gives no rows, so the peer's row is 0, as a status naming no row is today.
- `recipeRow` keeps mapping a retired producer name onto its successor.

Tests: `peersFakeDeps` answers rows per task: darn's for `darn-peer`, pawl's for a new off-machine project entry `pawl-peer` with an empty `recipe` field and status `running · Parent-Align`, tapestry's otherwise. `TestPeersVerb` asserts `darn-peer`'s row is Darn-Write's position in the darn recipe and `pawl-peer`'s is Parent-Align's in the pawl recipe. `TestHubPeersDeps_OverRealHub` reaches each branch of `peerRecipe` through the real `rows` seam: a peer pair whose seed records darn is read against darn's rows, and a peer pair whose seed records an empty recipe against tapestry's; with no worktree on this machine, a project entry with an empty `recipe` field is read against pawl's rows, an entry whose `recipe` is `darn` against darn's, and an entry with an empty `recipe` field against tapestry's.

**ImpactSummary:** The peers listing's row lookup becomes per peer; only the row numbers of darn and pawl peers change, and the verb's shape and other fields stay.
