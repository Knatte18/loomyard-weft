# Tuned loom defaults in the template; hub base layer goes sparse

The loom setup tuned in the loomyard hub becomes lyx's embedded template, so a new hub starts with it and no orch copies values by hand.
A hub's base layer now holds only what that hub tuned, so a later template change reaches every hub with no migration.
Hubs seeded before this change are migrated once, by the hub reconcile walk.

## What changes

Template values (`loom`, `shed`, `darn`, `claude`, `orch`, `webster`, `batcher`):

- `loom`: `twill-discussion` and `twill-plan` lenses, the reviewer and fixer fans, the reviewer and fixer models, `driver: haiku[medium]`, `discussion_max_bounces: 2`, `webster_max_bounces: 1`, `webster_auto_accept: true`.
- `shed`: `driver_kind: go`.
- `darn`: reviewer and fixer models and fans.
- `claude`: `prompt_cache_ttl_roles` of `driver: 5m`, `webster: 1h`, `orch: 1h`.
- `orch`: `soft_threshold_tokens: 150000`, `threshold_tokens: 200000`.
- `webster`: `kickstart_pack: false`.
- `batcher`: the `cautious` profile's `budget: 200000`.

Every fan and lens these values name exists in the `lenses` template, so a new hub resolves them with no `lenses.yaml` entry.
Loomyard's own values stay in its hub files: `board` kinds and labels, `darn.verify`, `landing.publish_verify`, `landing.deploy`, added `lenses`, `gate.slots` and `language.skills`.
`models` and `profiles` stay seed-only and full, since their file is the operator's registry.

Sparse base layer:

- In a hub, the base file of every module that is not seed-only holds only the units the hub tuned.
  A base with no unit keeps its file, holding a header comment, and loads as a present base with no units.
- Every write of a hub base stays sparse: seeding, key moves, `lyx config <module> --set` and the editor.
- `Load` no longer logs the template fill of a key a hub base lacks.
- `configengine`, `configsync`, `configcli` and `hubreconcile` docs and `PATTERN-config-strictness` state the sparse base; `PATTERN-sparse-hub-base` gains the write rule.

Migration:

- `hubreconcile`'s walk strips each present base file once, behind a tracked marker in `<BoardDir>/_lyx/config/` committed with the walk's board commit.
  It drops units equal to the current template or to the former value this change replaced; a unit with any other value is kept.
- The former values live in one table in `configsync`.
- The same walk drops former values from worktree files not yet marked sparse, including a pair forked before the migration.
- `lyx fabric clone` writes the marker itself, in its own board commit, when it seeded the board's base files from nothing.
- `lyx fabric reconcile` and `lyx config reconcile --apply` strip no present base and write no marker; the dry run reports the strip as pending.
- A failed walk restores the board's config files and removes a marker the pass wrote.
  The start verbs warn with the way forward and go on, through `hubreconcile.EnsureOrWarn`.

## Design decisions to know

- The tuned values weaken the review guards: Webster-Review gets one judged round and accepts a harvested round at the spent budget with its open findings, BLOCKING ones included; Discussion-Review gets two rounds; Webster, Merge and Darn reviewers and fixers drop to `sonnet[medium]`; an llm-driven run's driver drops to `haiku[medium]`.
  Under `driver_kind: go` no run uses the llm driver unless a hub, a driver override, run settings or `--driver` selects it.
  A hub pins any value back with `lyx config <module> --layer default --set`.
- The migration drops a base unit equal to the template or a former value even when a hub set it on purpose, such as `driver_kind: llm` or `kickstart_pack: true`, since a full base cannot tell a seeded value from a tuned one.
  The hub pins it again after the migration.
- The strip is one-time so a later deliberate value is never stripped again; removing the marker by a board commit runs it again.
- The config registry fingerprint covers the template text, so every hub's build stamp goes stale and its next start verb runs the walk.
- A machine running an older binary refills a stripped base with old template keys through its walk, `lyx fabric reconcile`, `lyx config reconcile --apply --layer default`, `lyx fabric clone` or `--layer default --set`.
  Run `update-plugins.sh` on every machine that uses the hub.
- A hub whose `batcher.profiles` differs from both the current and the former whole map keeps its profiles, so the `cautious` budget stays at its old value; edit it with `lyx config batcher --layer default`.
- A pair running during its hub's walk reads the new defaults from its next load on.
- A future language lens set must keep the fan names the loom defaults name.
- A task that retires or moves a key in the former-values table must update the table.
- `test-tiers` also changes `configsync` and the gate templates; whichever lands second merges over the other.

## To check by hand

- `PATTERN-sparse-hub-base` and the entries it sits among: the entry's length against `MaxEntryLineChars` (raised to 360 in `internal/pattern/check.go`), and that the eleven `PATTERN.md` entries shortened to fit the overview size now read as the parent's text, with no declared change missing.
- Run `lyx gate test --tags integration ./...` before pushing; this description does not record that run.
- After deploy, on one real hub (loomyard, quarry or NORCE Models), confirm the walk leaves each base holding only its tuned units, with the marker in the board commit.
