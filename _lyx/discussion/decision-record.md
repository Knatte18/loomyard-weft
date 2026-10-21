# Decision record — tuned-defaults-template

## Goal

The loom setup the operator tuned in the loomyard hub becomes lyx's embedded template, so every new hub starts with it and no orch copies values by hand.
Hubs cloned before the change follow the new template too, and every hub's base layer afterwards holds only what that hub tuned, so a later template change reaches every hub with no migration.

## Scope

In:

- The embedded templates of `loom`, `shed`, `darn`, `claude`, `orch`, `webster` and `batcher` take the loomyard values the board entry's split names as general preference (D1).
- A hub's base layer, the hub file of a hub-wide module and the repo default of a per-worktree module under `<BoardDir>/_lyx/config/`, becomes sparse over the template (D2).
- A one-time strip in the hub reconcile walk makes the base layers of hubs seeded before the change sparse, and drops the replaced template values from worktree files not yet migrated (D3).
- Every write of a hub base stays sparse: seeding, key moves, `--set` and the editor (D4).
- Load stops logging the template fill of a key a hub base lacks (D5).
- The module docs and the pattern background that state the base stays full (D6).

Out:

- Seed-only modules, `models` and `profiles`: their file is the operator's registry, seeded whole and never rewritten, and stays so.
- The standalone base layer, a worktree file with no board dir: it stays full and reconciled as today, since no hub reads it.
- The values the split keeps as loomyard's own: `board`'s kinds and labels, `darn.verify`, `landing.publish_verify` and `landing.deploy`, `lenses`' added entries, `gate.slots` and `language.skills`.
  They stay in loomyard's hub files because they differ from the template, with no edit by this task.
- Per-language lens sets (board entry `lenses-language-sets`) and the removal of `kickstart_pack` (board entry `kickstart-pack-remove`).
- No hand edit of any hub's `_board/_lyx/config/`: loomyard, quarry and NORCE Models are stripped by the walk of D3.
- A later tuning meant for every hub, from an orch or from `review-setup-audit`, is a template change in a task of its own; no mechanism here carries a hub-layer tuning into the template.

## Decisions

### D1 — The tuned values move into the templates

Each template below takes the value loomyard's hub layer holds today, and the key's template comment is rewritten to describe the new default instead of the old one:

- `loom`: `discussion_lenses: twill-discussion`, `plan_lenses: twill-plan`;
  `discussion_reviewer_model: ["opus[medium]", "opus[medium]", "sonnet[high]"]`, `plan_reviewer_model: ["opus[medium]", "sonnet[high]"]`;
  `webster_reviewer_model`, `webster_fixer_model`, `merge_reviewer_model` and `merge_fixer_model`: `sonnet[medium]`;
  `discussion_reviewer_fan: ["review-discussion", ""]`, `plan_reviewer_fan: ["review-plan", ""]`, `webster_reviewer_fan` and `merge_reviewer_fan: ["review-code", ""]`;
  `discussion_fixer_fan: fix-discussion`, `plan_fixer_fan: fix-plan`, `webster_fixer_fan` and `merge_fixer_fan: fix-code`;
  `driver: haiku[medium]`, `discussion_max_bounces: 2`, `webster_max_bounces: 1`, `webster_auto_accept: true`.
- `shed`: `driver_kind: go`.
- `darn`: `darn_reviewer_model` and `darn_fixer_model: sonnet[medium]`, `darn_reviewer_fan: ["review-code", ""]`, `darn_fixer_fan: fix-code`.
- `claude`: `prompt_cache_ttl_roles` becomes `driver: 5m`, `webster: 1h`, `orch: 1h`.
- `orch`: `soft_threshold_tokens: 150000`, `threshold_tokens: 200000`.
- `webster`: `kickstart_pack: false`.
- `batcher`: the `cautious` profile's `budget: 200000`.

Every fan and lens these values name must exist in the `lenses` template, as all of them do today, so a new hub resolves them with no hub `lenses.yaml` entry.

Rationale: the board entry's operator decision and split; the values are read from loomyard's hub files, which are the source.

### D2 — A hub's base layer is sparse over the template

In a hub, every file layer of a module that is not seed-only is sparse, the base layer included: the base file holds only the units the hub tuned, and the template is the one full layer.
A unit is what the module's stack already compares and replaces whole: a leaf key, a whole open map, a whole per-name entry.
The base file still exists, since strictness and the build stamp's absent-default check read its presence;
a base with no tuned unit holds only a header comment saying it holds the hub's tuned values over lyx's template, and loads, under `Load` and `LoadOrTemplate`, as a present base with no units.
`ReconcileHubWideAt` no longer adds a template key to a base file; it still runs the module's `Migrate` hook and removes keys the template lacks, as today.
A unit set in the base file equal to the template stays, as a pin, once the base is migrated (D3);
`lenses` keeps its own rule, dropping an entry equal to the template on every reconcile.
Every check that compares a file with the hub file, the leftover hub-wide copy's `Retired` and `Divergent` reports and the worktree strip's `Inherited` report among them, compares with the resolved template and hub file, never the raw hub file.

Rationale: a full base pins every template value at its seed time, so a template change never reaches a seeded hub; that is the failure the board entry describes.
The worktree layers already work this way, and the `lenses` module's base already holds only its tuned names.

### D3 — A one-time strip in the walk migrates existing hubs

The strip runs only in `hubreconcile`'s walk, behind a walk-only reconcile option as the worktree strip is behind `Migrate`, and only while the board's base-sparse marker is absent:

- Each present non-seed-only base file at the board dir drops a unit whose value, after the module's `Migrate` hook and after the key moves, equals the current template's or the former value D1 replaced.
- In the same walk, the pending strip of a worktree without the `sparse` marker also drops a unit equal to a former value, so a pair forked before the migration does not keep an old neutral value as an override.
- The former values live in one table in `configsync`, beside the legacy-module table, keyed by module and dotted unit, holding the D1 keys' pre-change template values;
  for an open map, `claude.prompt_cache_ttl_roles` and `batcher.profiles`, the unit and its former value are the whole map.
- The marker is a tracked file in `<BoardDir>/_lyx/config/`, distinct from the worktree `sparse` marker, written after the files, and only when every registered module's base file exists after the pass.
  It is part of the walk's board commit, and a failed walk removes a marker the pass wrote, beside restoring the board's config files to their prior bytes.
- Every other caller of `ReconcileHubWideAt`, `lyx fabric reconcile`, `lyx config reconcile --apply` and `lyx fabric clone`, strips nothing and writes no marker, and the dry run reports the base strip as pending per module with the keys it would drop.
  `lyx fabric clone` alone writes the marker, in its own board commit, when the board held no base file before its pass, since every file it seeds is already sparse (D4).
- Once the marker exists, no reconcile strips a base file again, so a value later set equal to the template or to a former value stays.

Bound: the strip drops only units equal to the current template or to a table value, so a unit holding any other value is kept;
it runs only in the walk, only on a board without the marker, and the table holds only the keys D1 changed.
A marked worktree's file is never stripped.

Rationale: an old seeded base holds the old neutral values, such as `discussion_lenses: ""`, and equality with the new template alone would keep them overriding it.
The config registry's fingerprint covers the template text, so every hub's build stamp goes stale and its next start verb runs the walk, which reaches loomyard, quarry and NORCE Models.
Keeping the strip and the marker in the walk keeps one writer and one commit for the marker, so no caller leaves it uncommitted on Bolt.
The one-time gate keeps a later deliberate value, such as `driver_kind: llm`, from being stripped by every later walk.

### D4 — Every write of a hub base stays sparse

In a hub, every path that writes a non-seed-only base file writes it sparse:

- a base seeded from the template holds no unit;
- a base seeded from the prime's copy holds that copy's units that differ from both the current template and the former values;
- a key move into a base writes only the moved units, never starts an absent target from the template or from the prime's full copy, and drops a moved unit equal to the template;
- `lyx config <module> --set` on the base writes only the keys set, through the sparse write the upper layers use, and an absent base file is created holding only them;
- the editor scaffolds an absent base file empty, as it does an upper layer; its empty-save abort is unchanged, so a base is emptied by a save that keeps the header comment.

A JSON-entries module keeps its own whole-entry write.

Rationale: a base written full by any one path pins the template again and undoes D2 once the marker is present, so D3 would never repair it.

### D5 — The template fill of a hub base is not logged

Load stops logging, at `Info`, the template keys it fills when `Board` is told and the module is not seed-only, as it already stops for per-name map names;
a standalone stack keeps the log.
The unknown-keys warning and every refusal stay as they are.

Rationale: with a sparse base every load fills most keys, and a log line per load would be noise.

### D6 — Docs state the sparse base

`configengine`'s, `configsync`'s, `configcli`'s and `hubreconcile`'s package docs, `configreg.Module`'s comments where they speak of the full base, and the `PATTERN-config-strictness` background drop "the base layer stays full" and state D2 to D5.

Rationale: `CLAUDE.md` puts the docs in the same commit.

## Constraints

- `PATTERN-config-layers`: only `configengine` reads, stacks and locates layer files; the strip's comparison with the template goes through it or `configsync`'s existing strip, never a new reader.
- `PATTERN-config-strictness`: `Load` and `LoadOrTemplate` still differ only on an absent base `_lyx/` or file; a present key-less base is a present base.
- `PATTERN-binary-change-reconcile`: the strip reaches existing hubs only through `hubreconcile.Ensure`'s walk.
- `PATTERN-hub-wide-config`: the base files and the marker stay at `<BoardDir>/_lyx/config/`, and the machine layer is untouched; no reconcile reads, rewrites or removes a machine file.
- `PATTERN-durable-vs-ephemeral-state` and `PATTERN-lyxdirs-single-declarer`: the marker is tracked under `_lyx`, its path declared where the config paths are.
- `PATTERN-comment-line-breaks`: every Go comment written one sentence per line.

## Auto-mode assumptions

- A unit equal to the template is dropped at migration even when a hub set it on purpose; the hub can pin it again with `--set` once migrated.
- A unit set equal to a former value on an existing hub between the deploy and its first walk is dropped by that walk.
- The former-values table holds only the D1 keys' values immediately before this change; a value from an older template generation is kept as tuned.
- Loomyard's `models.yaml` and `profiles.yaml` stay full, against the board's "holds only the kept entries", because seed-only files are the operator's registry.

## Open risks

- A machine still running an older binary that walks the shared board refills the stripped base files with template keys, old values included, and the marker keeps the new binary from stripping them again; the way forward is running `update-plugins.sh` on every machine that walks the hub, and `machine-handover` is the planned work on machines sharing one hub.
- A pair running when its hub's walk strips the base reads the new defaults from its next load on, such as reviewer models and fans between rounds; `driver_kind` is read once at seed and does not change.
- A future language lens set (`lenses-language-sets`) must keep the fan names D1's loom defaults name, or a hub seeded with that set fails to resolve them.
- A task that retires or moves a D1 key, such as `kickstart-pack-remove` or `shed-arbiter-all`, leaves a former-values entry naming a key the template lacks; whichever lands second updates the table, which the acceptance criterion on the table catches.
- `test-tiers` changes `configsync` and the gate templates concurrently; whichever lands second merges over the other's reconcile changes.

## Acceptance criteria

- A hub seeded with the new binary resolves every D1 value with no hub-file entry for it, its non-seed-only base files hold no unit, and its marker is committed by the clone.
- A board whose base files hold the pre-change template values in full ends a walk with each of those files holding no unit, the marker committed in the board commit, and every module resolving the new template's values.
- An unmarked worktree file holding a former value ends the same walk without it.
- A base unit holding a value equal to neither the current template nor a former value survives the strip, such as loomyard's `darn.verify` and `board` labels.
- A base unit set equal to a former value after the marker exists survives the next walk.
- `lyx fabric reconcile` and `lyx config reconcile --apply` on an unmigrated board strip nothing and write no marker, and the dry run reports the strip as pending.
- A failed walk leaves the board's config files at their prior bytes and no marker the pass wrote.
- `--set` on a hub base, the editor on an absent hub base and a key move into an absent hub base write only the units given or moved.
- A key-less base loads under `Load` and `LoadOrTemplate` without the absent-base refusal and without the template-fill log line.
- Seed-only modules' files and the standalone base are reconciled as before.
- Each former-values entry names a module and unit the template holds, with a value that differs from the current template's.
- A sparse hub file yields no false `Divergent` or `Retired` report for a leftover hub-wide copy.

## Notes for the plan writer

- The worktree sparse migration's `stripToSparse` already drops units equal to the layers below; the base strip is the same comparison with the template, plus the former values, as the layer below.
- `moveKeys`' base branch that starts an absent target from the template, and its overwrite of a template-default target, are what D4's key-move bullet changes.
- `readLayers` drops a comment-only file only for the machine and local layers; a comment-only base must reach the stack as a present, empty document.
- Tests and fixtures that assert a seeded base holds every template key, `hubforge`'s included, change with D2 and D4.
