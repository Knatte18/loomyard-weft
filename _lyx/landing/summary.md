# Tuned loom defaults become the template; hub base layers go sparse

The loom setup the operator tuned in the loomyard hub becomes lyx's embedded template, so every new hub starts with it and no orch copies values by hand.
A hub's base layer now holds only the units that hub tuned, so a later template change reaches every hub with no migration.

## What changes

Template values, as loomyard's hub files held them:

- `loom`: the twill discussion and plan lenses, the reviewer and fixer models, the review and fix fans for discussion, plan, webster and merge, `driver: haiku[medium]`, `discussion_max_bounces: 2`, `webster_max_bounces: 1` and `webster_auto_accept: true`.
- `shed`: `driver_kind: go`.
- `darn`: reviewer and fixer models and fans.
- `claude`: `prompt_cache_ttl_roles` of `driver: 5m`, `webster: 1h`, `orch: 1h`.
- `orch`: `soft_threshold_tokens: 150000` and `threshold_tokens: 200000`.
- `webster`: `kickstart_pack: false`.
- `batcher`: the `cautious` profile's `budget: 200000`.

Sparse hub base:

- In a hub, the base layer of every module that is not seed-only (the hub file of a hub-wide module, the repo default of a per-worktree module) holds only tuned units.
  A base with no tuned unit is a header-only file, present and key-less; `Load` and `LoadOrTemplate` treat it as a present base.
- Every base write stays sparse: seeding, key moves, `lyx config <module> --set` and the editor.
  `lyx fabric clone` writes a board marker in its own board commit when it seeds the base files from nothing.
- `ReconcileHubWideAt` no longer adds template keys to a base; it still runs `Migrate` and removes keys the template lacks.
- `Load` no longer logs the template fill of a hub base.
- Hubs seeded before this change are migrated once by `hubreconcile`'s walk: while the board marker is absent, each base file drops units equal to the current template or to the former value this change replaced (a table in `configsync`).
  The walk then commits the marker with the board commit.
  Worktree files without their own `sparse` marker also drop units equal to a former value, in the walk and in the single-pair call.
- A failed reconcile no longer stops a start verb: `hubreconcile.EnsureOrWarn` warns with the way forward and the verb goes on.
- Seed-only modules (`models`, `profiles`) and the standalone base layer (a worktree file with no board dir) stay full and reconcile as before.

Docs in the same change: the package docs of `configengine`, `configsync`, `configcli`, `hubreconcile`, `yamlengine` and others, `docs/overview.md`, `PATTERN-config-strictness` and the new `PATTERN-sparse-hub-base` entry.

## Design decisions to know

- The new loom values weaken the review guards.
  Webster-Review gets one judged round and accepts a harvested round at the spent budget with its open findings, BLOCKING ones included.
  Discussion-Review gets two rounds, and Webster-, Merge- and Darn-Review use `sonnet[medium]` reviewers and fixers.
  The operator's split chose these values; the verify gates, the PR-Gate alignment approve and the parent still check the code.
  A hub pins any value back with `lyx config <module> --layer default --set`.
- A unit equal to the template or a former value is dropped at migration even when a hub set it on purpose, such as `driver_kind: llm` or `kickstart_pack: true`, since a full base cannot tell a seeded value from a tuned one.
  The hub pins it again after migration, and once the marker exists no reconcile strips a base again.
- A hub whose base was refilled by an older binary (its walk, `lyx fabric reconcile`, `lyx config reconcile --apply --layer default` or a `--layer default --set` rewrite the base from the old template) is repaired by removing the board marker with a board commit.
  The next start verb's walk then strips again.
  The repair covers only values from the template immediately before this change.
- loomyard's own entries stay in its hub files untouched: `board` kinds and labels, `darn.verify`, `landing.publish_verify`, `landing.deploy`, `lenses`' added entries, `gate.slots` and `language.skills`.

## Check by hand

- Webster-Review flagged `PATTERN.md` edits no declared change listed (`undeclared-pattern-entry-edits`, BLOCKING).
  Later commits restored the ten entries and then shortened entries to fit the file's headroom, moving the detail into `pattern/` background files, and the diff against `main` shows those shortened entries.
  Check that each shortened entry in `PATTERN.md` keeps its invariant, with the cut clause present in its background file.
  `pattern/PATTERN-recipe-setting-lookup.md` is a new background file created by that move, and the spot checks of `PATTERN-edit-directive`, `PATTERN-push-both-sides` and `PATTERN-durable-vs-ephemeral-state` found the cut clauses in their background files.
- Nothing forces a board commit failure in the `lyx fabric clone` test, so the cleanup that removes the marker when that commit fails is untested (`internal/fabriccli/clone.go`).
- No run of `lyx gate test --tags integration ./...` is recorded for this change; run it before pushing to `main`.
- Deploying runs the walk on every hub at its next start verb; check loomyard, quarry and NORCE Models end it with key-less or sparse base files and the committed marker.
- Concurrent `test-tiers` work changes `configsync` and the gate templates; whichever lands second merges over the other's reconcile changes.
- A task that retires or moves a key in the former-values table (`kickstart-pack-remove`, `shed-arbiter-all`) must update the table.
- A future per-language lens set must keep the fan names the loom defaults name.
