# Rename the loom recipe to tapestry with a rename program

The recipe that `loom` named is now `tapestry`, so every `loom` left in the tree names the general run that `run-verbs-shared` builds.
No behaviour changes: rows, routing, config keys, verbs and gates stay as they were, bar the names listed below.

## What changes

- The recipe name is `tapestry`: the value a seed, a board entry's `recipe` field, a run profile's recipe block and a `defaults` entry carry, and the Go constant in `internal/shedrun` that declares it.
- `contracts/recipes/loom-recipe.yaml` is now `tapestry-recipe.yaml`, `recipes.LoomRecipe` is `recipes.TapestryRecipe`, and `loomrecipe.LoomRetiredRows` is `loomrecipe.TapestryRetiredRows`.
- The `Loom-Preflight` row, the one row only this recipe runs whose name says loom, is `Tapestry-Preflight`, with its producer, constants and file.
  Row names are durable resume identities, so `TapestryRetiredRows` maps `Loom-Preflight` onto it and a run parked on the old row resumes on the new one.
- A board entry with no `recipe` runs `tapestry`, as it ran loom before; the board README's Recipe cell shows `tapestry` for it.
- Prose that says "loom" for the variant ("loom run", "loom recipe", "loom and darn") now says tapestry, in comments, stencils, specs, docs, refusal texts, the README and the sandbox suites.
- `tools/tapestryrename` is the program that made the rename, kept in the tree as `tools/wordswap` is.
  It holds a table of file moves and a table of whole-token substitutions, takes `-dry-run` and `-root`, skips fixed path exclusions, and gofmts each Go file it changes.
  Its tests pin that it substitutes a listed token, leaves an unlisted `loom` alone and skips an excluded path.

## Design decisions

- **Only the variant's own names move.**
  What darn or pawl also use stays `loom`: the `lym loom ...` verbs, `internal/loomcli`, `loomengine`, `loomshed` and `loomrecipe`, the `loom` config module and `loom.yaml`, the `LOOM` program and driver strand names, the stencil directory `contracts/stencils/loom`, and the spec file names.
  The program therefore names each token it replaces instead of swapping the word.
- **No compatibility code for the old name.**
  The only run on the old name is the one that made this change, so nothing reads `loom` as a recipe name: `lym shed seed --recipe loom` is refused as an unknown recipe, and the refusal lists `tapestry`.
  The board entry's line about a migration lym runs itself is withdrawn; the one retired-row entry above is the only carried-over name.
- **Bolt is not in this diff.**
  The program never writes the board worktree, since a run edits only its own worktree.
  The board copies of stencils and specs take the new text through the existing stencil refresh; board entries and run records that carry `recipe` `loom`, and `_lym/config/loom.yaml`'s variant keys, are not rewritten here.
- **Out of scope, unchanged:** the board label `loom` and `board.yaml` label descriptions, and the measurement reports under `docs/research`, which record past runs.

Closes nothing: the board entry lists no issue.

## To check by hand

- `lym shed seed --recipe loom` is refused naming `tapestry`; `--recipe tapestry` seeds.
- A status file parked on `Loom-Preflight` resumes on `Tapestry-Preflight`.
- Board entries and run records on this hub that still carry `recipe` `loom` are renamed by hand, since nothing migrates them.
- `internal/loomrecipe/loomrecipe.go:18`: `TapestryRetiredRows` holds the `Loom-Preflight` entry and `TestRenameRetiredRows` in `internal/loomcli/wiring_darn_test.go` pins it.
- `internal/shedrun/seed_test.go:160`: a test asserts `loom` is refused as an unknown recipe and the refusal lists `tapestry`; the same for a `--recipe loom` call in `internal/shedcli/cli_test.go`.
- `README.md:58` and `docs/overview.md:650` name `tapestry` as the default recipe and the meaning of an empty recipe.
- `tools/sandbox/SANDBOX-CORE-SUITE.md:250`: the seed uses recipe `tapestry`, and no other spaced seed form under `tools/sandbox` keeps `loom`.
- `internal/battencli/arm.go:203`: the comment names `tapestry` as the prime seed's recipe value.
- `contracts/specs/shed-recipe-spec.md:112-113` say tapestry's row list and constructors, with `internal/loomrecipe` and `loomshed` names kept.
- Tests that use a recipe name in board and run-profile data (`internal/boardengine/task_test.go:187`, `internal/runprofile/runprofile_test.go:56-61` and the other recipe-keyed maps) say `tapestry`, while config module keys stay `loom`.
- `internal/lenses/doc.go:1` and `internal/reviewbudget/doc.go` name the `loom` config module where the shared fan key grammar is meant, and `tapestry` only where the recipe is meant.
