# The loom recipe is named tapestry, so loom is left to the general run

Today's loom recipe is renamed `tapestry`, with no change of behaviour.
Every `loom` left afterwards names something darn or pawl uses as well, which the follow-up `run-verbs-shared` turns into the general run.

## What is renamed

- The recipe name: `shedrun.RecipeTapestry` is `"tapestry"`.
  This covers seeds, the board's `recipe` field, `shedcli`'s arming table, run-profile recipe blocks and their `defaults`, `--recipe` examples and help texts.
- The recipe file `contracts/recipes/tapestry-recipe.yaml` and its Go names: `recipes.TapestryRecipe` and `loomrecipe.TapestryRetiredRows`.
- The row `Loom-Preflight`, now `Tapestry-Preflight`, with its engine name, constants, producer and `internal/loomshed/tapestrypreflight.go`.
- Prose that says loom for the variant: "loom run", "loom recipe", "loom and darn" and similar, in comments, stencils, specs and docs.
- An empty board `recipe` still runs the default variant, now `tapestry`, and the board README's Recipe cell shows `tapestry`.

## What stays loom

The `lym loom` verbs, the packages `loomcli`, `loomengine`, `loomshed` and `loomrecipe`, the `loom.yaml` module and its keys, the `LOOM` program, the driver strand, and the `loom` stencil directory and spec file names.
All of them serve darn or pawl as well.
Prose about verbs that serve every recipe, such as the help of `lym loom start` and `drive`, says "run" or "task" instead.
The measurement reports under `docs/research` are left as written.

## How

`tools/tapestryrename` holds the rename as data: a table of file moves, whole-token substitutions of identifiers and prose phrases, and recipe-name value fragments.
Each value fragment carries the context that tells the recipe name from the `loom` config module, such as `--recipe loom`, `"recipe":"loom"` or a test helper that takes a recipe, so `LoadConfig(dirs, "loom")` and a `run_settings` `loom` key stay.
No phrase is substituted right after `lym `, so a verb such as `lym loom run` keeps its name, and each Go file the program changes is formatted.
Its commit is followed by a commit of its output, unedited, so that diff can be checked against the tables.
The two commits after it are judgement only:
- the recipe value itself and the default recipe constants, the profile template among them, with the docs that state the value;
- help texts of verbs that serve every recipe, which the program's phrases had wrongly given "tapestry";
- spec lines that said "loom's" for the variant's own rows.

## Design decisions

- What is renamed and what stays follows one test: whether darn or pawl also uses it.
  By that test `loom.yaml` stays, although the board entry names it as variant config, since darn and pawl read its Code-Review, driver and friction keys.
- No code accepts the old name: a seed, board entry or profile that says `loom` is no longer read as `tapestry`, and `lym shed seed --recipe loom` is refused as unknown.
  The board entry's line about a migration lym runs itself is withdrawn by the parent's recorded decision, since no run on the old name outlives this one.
- The program never writes the board's records (Bolt).
  The stencil, spec and orch-rule copies there take the new text through the existing stencil refresh.

## After landing

When the new binary is deployed, the orch edits the hub's `profiles.yaml` with `lym config profiles`, by the parent's decision.
Every recipe key `loom` becomes `tapestry`, `defaults` included, and the inner `loom:` module key stays.
Until then, a tapestry task finds no profile block and runs on the hub's own config.
