```yaml
lens: code-correctness
outcome: reported
```

**Cases run, per function or branch**

- `Outputs.recipeOf`:
  - Empty `Recipe` with `DefaultRecipe` `"darn"` gives `"darn"`. Correct.
  - `Recipe` `"tapestry"` gives `"tapestry"`. Correct.
  - Empty `Recipe` with an empty `DefaultRecipe` gives `""`. That is the same as the old `Board.RecipeOf`. A real hub cannot reach it, because `LoadConfig` fills the template's `darn`. A path-only `Config`, such as the one in `ReadRunSettings`, never resolves a recipe. Correct.
- `Board.RecipeOf`: it delegates to `b.out.recipeOf`. It returns the same values as before for the three inputs above. Correct.
- `recipeCell`:
  - An empty recipe renders `cellText(default)`, and a named recipe renders `cellText(Recipe)`. This matches the old branch.
  - The profile suffix path is unchanged.
  - The value receiver copies `Outputs` and its `Types` slice header, which is harmless.
  - Both call sites pass `b.out`, the same `Outputs` the old code read `DefaultRecipe` from. Correct.
- `ReadBoardType` closure (not changed in the diff):
  - A failing `OpenHub` returns the error with an empty string.
  - A failing `GetTask` returns the error.
  - A missing task returns the "not found" error.
  - None of these is swallowed.
  - I did not read `seamchild.go:65`, so I did not check the summary's claim that these errors become a Stuck verdict. That claim is the code-affected-set and handler lenses' to check.
- `peerRecipe` and the `hubPeersDeps` board closure:
  - Every task that reaches `peerRecipe` comes from the `board` closure. `peers.go:113` and `peers.go:170` call `deps.board()`, `deps.rows(task)` at `peers.go:132` and `peers.go:147` receives those tasks, and `ListTasksFull` at `peers.go:290` is the only source. The closure resolves `Recipe` with `board.RecipeOf` at `peers.go:295`.
  - An empty board recipe therefore reaches `peerRecipe` as the default (`darn`), and the doc change is accurate.
  - A project entry still gives `pawl`.
  - A self-run seed with an empty `Recipe` falls through to the tapestry fallback at `peers.go:259`. That matches the summary's claim.
  - `own_row` at `peers.go:147` uses the same path. Correct.
- `TestLoadConfig_DefaultRecipe` with `t.Parallel`:
  - `writeBoardConfig` creates its own `t.TempDir()`.
  - `LoadConfig` is given explicit `Dirs`, so I found no process-global state.
  - The `tiergate.Require` call comes before `t.Parallel`, as the other tests do. Correct.

**Findings**

None. The refactor preserves behavior on every input I ran. I found no swallowed error and no inconsistent state on a failure path.

**Tool calls**

1. `grep` over `internal/loomcli/*.go` for `peerRecipe`, `GetTask`, `ListTasksFull` and the `board`/`rows` callers, to confirm that every task given to `peerRecipe` is resolved by the board closure. My context held only `peers.go` lines 235–315.
