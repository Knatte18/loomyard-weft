# Spawned code roles load the hub's language skills

Spawned code roles (the darn writer, the Webster master and recovery strand, the Burler seats of code rounds and the merge conflict session) wrote and reviewed Go without any `scribe:golang-*` skill.
A new hub-wide config module, `language`, now names the skills every code role loads after its own list, so lyx hard-codes no language.
The darn writer also gains `scribe:testing`, which Webster and Burler already load.

Until the operator sets this hub's value, code roles load no language skill, as today.
After the post-deploy reconcile seeds `language.yaml`, run `lyx config language --set 'skills=[scribe:golang-comments, scribe:golang-testing]'` in the prime.

## What changes

- New package `internal/languageconfig`: module `language`, `HubWide`, file `<BoardDir>/_lyx/config/language.yaml`, one key `skills` (ordered list of skill names, default empty).
  It is registered in `configreg` in alphabetical order and joins the degrading callers in `pattern/PATTERN-config-strictness.md`.
- The darn writer, the Webster master and recovery strand, and the merge conflict session at Merge-In and at Finalize spawn with their own list followed by the language skills in config order.
  A skill repeated in the config or already in the role's own list appears once, at its first position.
- Burler profiles gain a boolean `language-skills`, accepted on recipe `BurlerReview` and `BurlerFix` rows and in `lyx burler run`'s profile YAML.
  When true, every seat of that half loads `burlerSkills` followed by the language list.
  It is set on the Webster, Merge and Darn review rows and their fix rows in `contracts/recipes/loom-recipe.yaml` and `contracts/recipes/darn-recipe.yaml`; the Discussion and Plan rows do not carry it.
- `darnSkills` becomes `scribe:prose`, `scribe:code-quality`, `scribe:testing`.
- `internal/loomcli` and `internal/webstercli` load the module once per process (`lyx webster recover-batch` included) and tell the list to the engines.
  `landingshed` loads no config; `wire()` passes the list through `landingshed.Deps` to `mergeresolve.Deps`.
- Docs: the touched packages' `doc.go`, `docs/overview.md`, and `contracts/specs/refusal-spec.md` (new `language.yaml` rows).

## Design decisions

- `skills` is classified Guard, so a worktree's local override cannot set it; the list reaches code-reviewing seats, and a pair must not be able to drop its reviewer's rules.
- Loading degrades: an absent `language.yaml`, or wiring with no board dir, yields the empty list and exactly today's skill lists, and no worktree `language.yaml` is ever read.
  A present file that does not parse, or an entry that is empty or has surrounding whitespace, fails the load naming `skills` and `lyx config language`; through `lyx webster` it refuses with `config_invalid`.
  The loader does not check that a named skill exists; shuttle skips an unknown one with a logged warning.
- The template comment steers the list toward comment and testing skills and away from build skills such as `scribe:golang-build`; code filters no name.
  A build skill sends agents to raw commands that the session deny (`rawGoCommandPattern`) refuses only for module-wide or tier runs, so a build skill listed in a hub that builds through `lyx gate` lets agents run, for example, `goimports -w` or a single-package `go test` outside the gate slots.
- Each listed skill adds to every code role's context on every spawn, and a misspelled one is skipped with only a logged warning.
- `burler-fixer-fork` plans to fork the fixer from the reviewer, which would inherit its language skills; that design decides whether the fix rows keep `language-skills`.

## To check by hand

- The `config file content invalid` and `config file unreadable` rows in `contracts/specs/refusal-spec.md` and the `lyx webster` master-stop wording in `internal/websterengine/doc.go`: a review left open that the card 6 handling of an unreadable `language.yaml` row and the doc text on the master stop may be stale.
- After landing and deploy, confirm `language.yaml` is seeded, set the value as above, and check that a spawned darn writer or Burler seat lists the language skills.
- `cmd/lyx/testdata/test-budget.yaml` changed with the new package's tests; check the budget rows are as intended.

No issues are closed by this change.
