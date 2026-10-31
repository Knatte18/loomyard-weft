# Decision record — tapestry-rename

## Goal

Today's loom recipe is renamed `tapestry`, mechanically and with no change of behaviour, so that every `loom` left afterwards names the general run that the follow-up task `run-verbs-shared` builds.
A Go program under `tools/` carries out the code rename, and lym itself migrates the records a hub already holds, on this hub and on every other.

## Scope

In scope, renamed to `tapestry`:

- The recipe name: the value `loom` a seed, a board entry's `recipe` field, a run profile's recipe block and a `defaults` entry carry, and the Go constant that declares it.
- The recipe's own file and Go names: `contracts/recipes/loom-recipe.yaml`, the embedded `recipes.LoomRecipe` byte var and `loomrecipe.LoomRetiredRows`.
- The `Loom-Preflight` row, the one row only the tapestry recipe runs whose name says loom, with its producer, constants and file.
- Prose that says "loom" for the variant: phrases such as "loom run", "loom recipe", "loom and darn", in code comments, stencils, specs, docs and refusal texts.

Out of scope, staying `loom` because they serve every recipe:

- The `lym loom ...` verbs, the packages `internal/loomcli`, `internal/loomengine`, `internal/loomshed` and `internal/loomrecipe`, and their Go names other than those listed above.
- The `loom` config module and its `loom.yaml`, which darn and pawl read for the Code-Review perch, the driver and the friction producer.
- The `LOOM` program name and the driver strand name.
- The stencil directory `contracts/stencils/loom`, its file names, and the spec file names under `contracts/specs`.
- The board label `loom` and the hub's own label descriptions in `board.yaml`, which the operator owns.
- The measurement reports under `docs/research`, which record past runs.

## Decisions

### D1 — The split between the variant and the general run

Only what the tapestry recipe alone uses, and whose name says loom, is renamed: the recipe name, the recipe file and its Go names, and the `Loom-Preflight` row.
Everything darn or pawl also reads stays `loom`, so `loom` is left on exactly the parts the general run will own.
Rationale: the board entry leaves this split to the run and names `internal/loomcli` as general because it already serves darn; the same test, whether darn or pawl uses it, sorts every other package, the config module, the stencil directory and the strand names.

### D2 — The rename program

A new program `tools/tapestryrename` (new) holds the rename as data: a table of file moves and a table of whole-token substitutions, identifier tokens and prose phrases, applied over the repository with fixed path exclusions.
It is run once from the repository root, its result is committed as its own commit, and hand edits follow in later commits.
It takes no flags beyond a dry run and the root, and it is kept in the tree afterwards as the record of the rename, as `tools/wordswap` is.
Rationale: the board entry asks for a program like `tools/wordswap`; a generic word swap cannot tell the variant's `loom` from the general one, so the program names each token it replaces instead of replacing the word.

### D3 — Records already written: lym migrates them on read

lym reads the old name as the new one wherever a record a hub already holds carries it, the way a seed recording `beam` reads as pawl:

- A seed whose recipe is `loom` reads as `tapestry`.
- A board entry whose `recipe` is `loom` reads as `tapestry`, and the next board write stores the new name.
- A run profile's recipe block named `loom`, and a `defaults` entry keyed `loom`, read as `tapestry`; when a file holds both, the `tapestry` one wins.
- A run's status file parked on the row `Loom-Preflight` is carried onto `Tapestry-Preflight` by the retired-row rename the loom wiring already applies.
- The board copies of stencils, specs and the orch rules take the new text through the existing stencil refresh, which rewrites an untouched copy from the shipped default.

Bound: each alias maps exactly one old value onto its successor and nothing else, so a mistyped recipe name is still refused as unknown.
Rationale: the board entry requires that another hub pick the rename up by a migration lym runs itself; a read-side alias needs no walk, no lock and no commit of its own, and the program never writes the board worktree, since a run edits only its own worktree.

### D4 — An empty recipe

A board entry with no `recipe` runs `tapestry`, as it ran loom before, and the board README's Recipe cell shows `tapestry` for it.
Rationale: no behaviour changes; the empty value keeps meaning the default variant, now under its new name.

### D5 — Strand and program names

The `LOOM` program and the driver strand keep their names.
Rationale: both serve darn and pawl runs as well, so by D1 they are the general run's.

### D6 — Input that names the old recipe

A verb that takes a recipe name, such as `lym shed seed --recipe`, accepts only the current vocabulary, so `loom` there is refused as an unknown recipe and the refusal lists `tapestry`.
Bound: only a record already written is migrated (D3); new input must use the new name.
Rationale: the alias exists so old records keep working, and accepting the old name for new input would keep it alive in new records.

### Added after Discussion (parent, 2026-10-31): No compatibility or migration code for the old loom name

Decision: Add no code that accepts the old name: no legacyRecipeLoom, no decodeSeed mapping of "loom" to tapestry, and no migration lym runs on another machine or hub. Rename the code and Bolt in this change and nothing more. The board body's line about a migration lym runs itself is withdrawn.

Rationale: The only run on the old name is this one, and it ends before the new binary is deployed; no other run or seed will meet the new binary under the old name (operator, 2026-10-31).

## Constraints

- No behaviour changes: rows, routing, config keys, verbs and gates stay as they are, bar the names D1 renames.
- `internal/shedrun` stays the sole declarer of the recipe-name vocabulary.
- `boardengine` imports no recipe package, so its alias and its default recipe name are its own constants.
- Row names are durable resume identities, so the `Loom-Preflight` rename goes with a retired-row entry.
- Work only in this worktree: the program never writes the board worktree.

## Auto-mode assumptions

- `loom.yaml` is the general run's config even though the board entry names it as an example of variant config, because darn and pawl read it; its discussion and plan keys move only when `run-verbs-shared` splits the module.
- "a loom run" in existing prose means the variant, since darn runs are called darn runs; the program rewrites it to "a tapestry run".
- `lym loom status` and similar verb names in prose stay, since the verbs keep their names.

## Open risks

- Prose that says "loom" for the variant in a phrase the program does not list stays `loom` until `run-verbs-shared` reads it.
- A hand-edited board copy of a stencil keeps its old text, as with any stencil change.

## Acceptance criteria

- D1: the recipe vocabulary holds `tapestry` in place of `loom`, and the general packages, `loom.yaml`, the stencil directory and the `lym loom` verbs keep their names.
- D2: `tools/tapestryrename` exists, holds its move and substitution tables, and a test pins that it substitutes a listed token, leaves an unlisted `loom` alone and skips an excluded path.
- D3: a seed recording `loom` reads as `tapestry`; a board entry with `recipe` `loom` reads as `tapestry`; a run profile block and a default keyed `loom` read as `tapestry`; a status parked on `Loom-Preflight` resumes on `Tapestry-Preflight`.
- D4: the board README shows `tapestry` for a task with no recipe, and Seed-Child seeds `tapestry` for it.
- D5: the `LOOM` program and the driver strand names are unchanged.
- D6: `lym shed seed --recipe loom` is refused as an unknown recipe naming `tapestry`.
- The gate's tests pass on the renamed tree.
