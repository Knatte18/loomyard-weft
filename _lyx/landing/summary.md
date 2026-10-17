# Lens fan and twill fixer for Webster, Merge and Darn review

Webster-Review, Merge-Review and Darn-Review can now run the Burler lens fan and the twill fixer, chosen per round, as Discussion-Review and Plan-Review already could.
Every new key defaults to empty, so no run changes until a hub sets one.

## What changes

- **New fan keys.**
  `loom.yaml` gains `webster_fan`, `webster_fix_fan`, `merge_fan` and `merge_fix_fan`.
  `darn.yaml` gains `review_fan` and `review_fix_fan`.
  Each takes the existing grammar: one fan name, or a per-round list whose last entry serves every later round, with `""` running that round solo or with the single fixer.
  All six are closed to the local layer and declared as open maps, so reconcile and `--set` carry a list whole.
  An empty list, a value of another shape and an entry naming an undefined fan are refused at load, naming the module, the key and the entry.
- **One fan grammar.**
  `FanList`, its shape error (now `burlerengine.FanShapeError`) and the fan validation (`burlerengine.ValidateFanKeys`) move from `loomengine` into `burlerengine`, and loom and darn both load through them.
  The lookup that names the key behind a malformed value moves to `configengine.KeyAtLine`.
- **Wiring.**
  `wire` adds the Webster and Merge Bouncer and review rows to `RowClusterFans`, and their review rows to `RowFixFans`.
  `wireDarnFans` adds the Darn rows from `darn.yaml`.
  `RowModelKeys` now has entries for the Webster, Merge and Darn review and fix rows.
  Merge and Darn name Webster's model keys (the new `ReviewSettings.WebsterKeys`), since they run on Webster's models.
  The Darn fixer fan key reads `review_fix_fan in darn.yaml`, because its model keys live in `loom.yaml`.
- **Read-only git for forks.**
  The fork boilerplates in `burler-cluster-fan.md` and `burler-fix-self-check.md` now allow git commands that only read, such as `git diff`, `git log` and `git show`, and forbid any that change the repository, its index, its refs or its working tree.
  The engine's fork audits are unchanged and still fail a fork that runs a mutating git subcommand.
- **Code fans in `burler.yaml`.**
  `review-code` runs `code-correctness`, `code-affected-set`, `code-tests` and `code-pattern-fit`.
  `fix-code` runs `fix-code-correctness` with the existing `fix-consistency`, `fix-proposal` and `fix-affected-set`.
  Each review lens names the rubric items it emphasizes by their headings, and qualifies the ones only some of the three code rubrics carry.
- **Wording.**
  The fan-conflict refusal in `shedrecipe`'s BurlerReview entry says the list came "from the hub config", since a darn row's list comes from `darn.yaml`.
  The recipe comments, `internal/loomengine/config.go`'s header, both config templates, the `burlerengine`, `darnengine` and `configcli` package docs and `docs/overview.md` no longer say these perches always run solo.

## Design decisions to know

- With a `RowModelKeys` fix fan key present, the Webster, Merge and Darn fix halves now need the fan names their review half recorded, as Discussion and Plan fix halves already do.
  So a round reviewed by an older binary and fixed by this one refuses, and the way forward is the existing one: `goto` the segment's review row and resume.
- The Darn entries are wired in `wireDarnFans`, beside the darn fan lists, so a darn row never needs recorded fans unless its review row records them.
- The gate-quiescence test now also checks the shipped darn recipe, and admits `DarnWrite` among the gated writer engines, since the darn writer's seat never authorizes forks.

## Check by hand

- The `review-code` and `fix-code` lens texts are a first draft and have not been benched; the 6-call fork cap may be tight against `git diff` over a large range.
- The read-only git permission rests on the stencil wording plus the audit's list of mutating subcommands; a state-changing git command outside that list would pass the audit.
- A hub `burler.yaml` that defines a lens named like a new one (`code-tests`, ...) shadows the new text through the per-name fallback.
- The hub's own `loom.yaml` and `darn.yaml` are unchanged; the orch sets the new keys after landing.
- Not run by hand: a real Webster, Merge or Darn round with a fan and a twill fixer.
