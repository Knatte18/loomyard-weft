# Decision record — tuned-defaults-template

## Goal

The loom setup the operator tuned in the loomyard hub becomes lyx's embedded template, so every new hub starts with it and no orch copies values by hand.
Hubs cloned before the change follow the new template too, and every hub's base layer afterwards holds only what that hub tuned, so a later template change reaches every hub with no migration.

## Scope

In:

- The embedded templates of `loom`, `shed`, `darn`, `claude`, `orch`, `webster` and `batcher` take the loomyard values the board entry's split names as general preference (D1).
- A hub's base layer, the hub file of a hub-wide module and the repo default of a per-worktree module under `<BoardDir>/_lyx/config/`, becomes sparse over the template (D2).
- A one-time strip in the hub reconcile walk makes the base layers of hubs seeded before the change sparse, and `hubreconcile`'s worktree strip drops the replaced template values from each worktree file not yet migrated (D3).
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

The loom values weaken the review guards:
Webster-Review gets one judged round instead of `review_max_bounces`' three and, since its Bouncer sets no `blocking_escalates`, accepts a harvested round at the spent budget with its open findings, BLOCKING ones included, instead of running the soft-cap rule;
Discussion-Review gets two rounds instead of three;
Webster-Review's, Merge-Review's and Darn-Review's reviewer drops from `sonnet[high]` to `sonnet[medium]`, and their fixer from `fixer_model`'s `opus[medium]` to `sonnet[medium]`;
Plan-Review's reviewer from its second round on, and Discussion-Review's from its third, drops from `reviewer_model`'s `opus[medium]` to `sonnet[high]`;
an llm-driven run's driver drops from `sonnet[medium]` to `haiku[medium]`, bounded by `shed`'s `driver_kind: go`, under which no run uses the llm driver unless a hub, a module's driver override, a task's run settings or the run's `--driver` selects `llm`, and a hub pins the model back with `lyx config loom --layer default --set driver=sonnet[medium]`.
So a change reviewed by fewer rounds and lighter models, with an open BLOCKING finding accepted at Webster-Review, can land.
Bound: the operator's split chose these values;
the verify gates, PR-Gate's alignment approve or the parent still check the task's code, while Merge-Review bounds none of it, since it reviews only the task's interaction with what the parent gained and is skipped when the parent gained nothing the task touches;
every value is a loom key a hub pins back with `lyx config loom --layer default --set`;
and an existing hub reaches them only through D3's strip of units equal to the former values.

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

The base strip runs only in `hubreconcile`'s walk, behind a walk-only reconcile option as the worktree strip is behind `Migrate`;
the worktree's former-value drop runs wherever `hubreconcile` strips a worktree:

- While the board's base-sparse marker is absent, each present non-seed-only base file at the board dir drops a unit whose value, after the module's `Migrate` hook and after the key moves, equals the current template's or the former value D1 replaced.
  A base file the strip, or the reconcile's removal of keys the template lacks, leaves with no unit is written as D2's header-only base and never deleted:
  the sparse-file reconcile the worktree strip uses removes an emptied file, and a deleted base would withhold the marker and keep the build stamp stale on every start verb.
- Every pending strip of a worktree without its own `sparse` marker, in the walk and in the single-pair call (`hubreconcile`'s `Options.Pair`), also drops a unit equal to a former value, whether or not the board marker exists, so a pair forked before the migration, a pair the migrating walk skipped as mid-merge and a clone's prime do not keep an old neutral value as an override.
  Its bound is the worktree's own `sparse` marker, written after the strip, so each worktree is stripped of former values once.
- The former values live in one table in `configsync`, beside the legacy-module table, keyed by module and dotted unit, holding the D1 keys' pre-change template values;
  for an open map, `claude.prompt_cache_ttl_roles` and `batcher.profiles`, the unit and its former value are the whole map.
- The marker is a tracked file in `<BoardDir>/_lyx/config/`, distinct from the worktree `sparse` marker, written after the files;
  the walk writes it only when every registered module's base file exists after the walk's pass.
  It is part of the walk's board commit, and a failed hub-wide step, a failed hub-wide reconcile or board commit, removes a marker the pass wrote, beside restoring the board's config files to their prior bytes, as `hubreconcile` already restores them on that failure.
  A failure in a later worktree step leaves the committed board and its marker in place, and the next walk strips no base again and retries the worktrees, which the worktree former-value drop above allows with the marker present.
- Every other caller of `ReconcileHubWideAt`, `lyx fabric reconcile`, `lyx config reconcile --apply` and `lyx fabric clone`, strips no present base file and writes no marker, and the dry run reports the base strip as pending per module with the keys it would drop.
  A seed of an absent base from the prime's copy writes only the units that differ from the template and, while the prime has no `sparse` marker, from the former values (D4), in every caller that seeds one:
  the walk and `lyx config reconcile --apply --layer default`, which pass `SeedDefaults`, and `lyx fabric reconcile` and `lyx fabric clone`, whose hub-wide modules with open maps or `SeedsFromPrime` seed from the prime's or the weft's base.
  That is the sparse seed of an absent file, not a strip of a present one, so `PATTERN-sparse-hub-base`'s "only the units the hub tuned" covers it.
- `lyx fabric clone` alone writes the marker, in its own board commit, when the board held no base file before the clone's pass, since every file it seeds is already sparse (D4);
  the walk's precondition that every base file exists does not apply to it.
  The clone's pass runs without `SeedDefaults` and leaves every per-worktree repo default absent, so a marker the clone writes before those defaults exist is intended, and the walk seeds them after the marker, sparse (D4).
- Once the marker exists, no reconcile strips a base file again, so a value later set equal to the template or to a former value stays.
- An absent board marker makes a build stamp naming the running build stale, as an absent repo default does, so a marker removed by hand with a board commit makes the next start verb's walk strip the base files again.

Bound: the strip drops only units equal to the current template or to a table value, so a unit holding any other value is kept;
the base strip runs only in the walk, only on a board without the marker, and the table holds only the keys D1 changed.
A marked worktree's file is never stripped.

Rationale: an old seeded base holds the old neutral values, such as `discussion_lenses: ""`, and equality with the new template alone would keep them overriding it.
The config registry's fingerprint covers the template text, so every hub's build stamp goes stale and its next start verb runs the walk, which reaches loomyard, quarry and NORCE Models.
Keeping the strip and the marker in the walk keeps one writer and one commit for the marker, so no caller leaves it uncommitted on Bolt.
The one-time gate keeps a later deliberate value, such as `driver_kind: llm`, from being stripped by every later walk.

### D4 — Every write of a hub base stays sparse

In a hub, every path that writes a non-seed-only base file writes it sparse:

- a base seeded from the template holds no unit;
- a base seeded from the prime's copy holds that copy's units that differ from the current template and, only while the prime worktree has no `sparse` marker of its own, from the former values;
  the gate is the prime's marker, not the board's, because the walk seeds a clone's repo defaults after the clone wrote the board marker and before it strips the prime;
- a key move into a base writes only the moved units and never starts an absent target from the template or from the prime's full copy;
  a target absent when a move reaches it, in a pass that seeds from the prime, takes the sparse prime seed of the two bullets above first, and the moved units are laid over it, while a pass that does not seed from the prime starts the target empty;
  a target unit equal to the template is overwritten by the moved value only while the board marker is absent, and is kept, as a pin, once it exists;
  a target unit that differs from the template, a prime-seeded one included, is kept and the moved value dropped, as today;
  a target the move leaves with no unit is written as the header-only base, never deleted,
  and a source base the move leaves with no unit is written as the header-only base too, never deleted;
  a key move writes every moved unit but one a target pin kept above overrides, and the walk's one-time base strip, which runs after the key moves (D3), drops a moved unit equal to the template or a former value on an unmigrated board, so no other caller drops one, bar `lenses`' template-equal entries (D2);
- `lyx config <module> --set` on the base writes only the keys set, through the sparse write the upper layers use, and an absent base file is created holding only them;
- the editor scaffolds an absent hub base with D2's header comment and no unit, so the header is present; its empty-save abort is unchanged, so a save that keeps the header leaves a key-less base.

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

### Added after Discussion (operator, 2026-10-21): Existing hubs follow the template automatically, once per template change

Decision: Each hub records a stamp of the template version its config was last reconciled against. When lyx reads a hub's config and the stamp differs from the binary's templates, it reconciles that hub once: it drops seeded default-layer entries equal to the old template, then writes the new stamp. A matching stamp costs one comparison and nothing else. If the reconcile fails, the hub's config is left as it was, the stamp is not updated, lyx goes on with the existing config, warns with the error and the command to reconcile by hand, and tries again at the next read. No hub needs a manual step after a new binary is deployed.

Rationale: The operator sees no reason to keep this manual; the stamp keeps the per-read check cheap, and a failed reconcile must never break a hub or leave a half-written config.

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
- A base unit equal to a former value is dropped at migration even when the hub set it on purpose before the deploy, such as `driver_kind: llm` or `kickstart_pack: true`, since a full base cannot tell a seeded value from a tuned one; the hub pins it again with `lyx config <module> --layer default --set` once migrated.
- An unmarked worktree unit equal to a former value is dropped even when the hub base holds another tuned value for that unit, so the worktree resolves to the base's value instead of the former one, since a full worktree file cannot tell a seeded copy from a tuned override; the worktree pins it again with `lyx config <module> --set`.
- The former-values table holds only the D1 keys' values immediately before this change; a value from an older template generation is kept as tuned.
- Loomyard's `models.yaml` and `profiles.yaml` stay full, against the board's "holds only the kept entries", because seed-only files are the operator's registry.

## Open risks

- A machine still running an older binary refills a stripped base with template keys, old values included, whatever its stamp says: its walk, `lyx fabric reconcile`, `lyx config reconcile --apply --layer default` and a `lyx fabric clone` of an existing weft run the old full reconcile, and `lyx config <module> --layer default --set` rewrites the module's base from the old template.
  The marker then keeps the new binary from stripping it again.
  Running `update-plugins.sh` on every machine that uses the hub prevents a refill, and `machine-handover` is the planned work on machines sharing one hub.
  A refilled base is repaired by removing the board marker by hand with a board commit, after which the next start verb's walk strips every base again (D3) and drops any pin equal to the template or a former value, which the hub sets again with `--set`.
  The repair is complete only against a binary from immediately before this change: a refilled value of an older template generation equals neither the template nor a former value, so it is kept as tuned and removed by hand.
- A hub whose `batcher.profiles` differs from both the current and the former whole map, by a profile of its own or an older template generation, keeps its profiles map, so the `cautious` budget stays at its old value; the way forward is editing that hub's base with `lyx config batcher --layer default`, setting the `cautious` profile's `budget` to 200000, since `--set` sets an open map's entry only whole.
- A pair running when its hub's walk strips the base reads the new defaults from its next load on, such as reviewer models and fans between rounds; `driver_kind` is read once at seed and does not change.
- A future language lens set (`lenses-language-sets`) must keep the fan names D1's loom defaults name, or a hub seeded with that set fails to resolve them.
- A task that retires or moves a D1 key, such as `kickstart-pack-remove` or `shed-arbiter-all`, leaves a former-values entry naming a key the template lacks; whichever lands second updates the table, which the acceptance criterion on the table catches.
- `test-tiers` changes `configsync` and the gate templates concurrently; whichever lands second merges over the other's reconcile changes.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - clone-marker-contradicts-marker-precondition (class consistency, severity MEDIUM, open in rounds 3)
  - prime-seed-drop-contradicts-strip-nothing-callers (class consistency, severity MEDIUM, open in rounds 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A hub seeded with the new binary resolves every D1 value with no hub-file entry for it, its non-seed-only base files hold no unit, and its marker is committed by the clone.
- A board whose base files hold the pre-change template values in full ends a walk with each of those files holding no unit, a base the strip empties being present and key-less, the marker committed in the board commit, and every module resolving the new template's values.
- An unmarked worktree file holding a former value ends the same walk without it, and so does an unmarked pair reconciled through `hubreconcile`'s `Options.Pair` on a board that already holds the marker.
- A hub cloned from a repo whose tracked worktree files are full and unmarked at the pre-change template values resolves every D1 value after the clone's walk, and a pair skipped as mid-merge in the migrating walk drops its former values in the next walk.
- A board whose marker is removed by a board commit, under a build stamp naming the running build, has its base files stripped by the next start verb's walk.
- A base unit holding a value equal to neither the current template nor a former value survives the strip, such as loomyard's `darn.verify` and `board` labels.
- A base unit set equal to a former value after the marker exists survives the next walk.
- `lyx fabric reconcile` and `lyx config reconcile --apply` on an unmigrated board strip no present base file and write no marker, and the dry run reports the strip as pending.
- A failed hub-wide step of a walk leaves the board's config files at their prior bytes and no marker the pass wrote.
- `--set` on a hub base, the editor on an absent hub base and a key move into an absent hub base write only the units given or moved, bar a move target's sparse prime seed in a pass that seeds from the prime.
  A key move out of a hub base holding only the moved unit leaves that base present and key-less.
- A key-less base loads under `Load` and `LoadOrTemplate` without the absent-base refusal and without the template-fill log line.
- Seed-only modules' files and the standalone base are reconciled as before.
- Each former-values entry names a module and unit the template holds, with a value that differs from the current template's.
- A sparse hub file yields no false `Divergent` or `Retired` report for a leftover hub-wide copy.

## PATTERN changes

- `PATTERN-sparse-hub-base` — adds: Writing a hub's base file (a hub-wide module's hub file or a per-worktree repo default, not seed-only): only the units the hub tuned, never a template fill; only `hubreconcile`'s one-time walk strip drops a unit the template holds, bar `lenses`' template-equal entries.

## Notes for the plan writer

- The worktree sparse migration's `stripToSparse` already drops units equal to the layers below; the base strip is the same comparison with the template, plus the former values, as the layer below.
- `moveKeys`' `loadTarget`, which starts an absent target from the prime's full copy and, failing that, from the template, its overwrite of a template-default target, and `moveFile.write`'s delete of an emptied hub base file, source or target, are what D4's key-move bullet changes.
- `readLayers` drops a comment-only file only for the machine and local layers; a comment-only base must reach the stack as a present, empty document.
- Tests and fixtures that assert a seeded base holds every template key, `hubforge`'s included, change with D2 and D4.
