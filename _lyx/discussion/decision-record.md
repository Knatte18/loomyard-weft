# Decision record — config-layers-run-settings

## Goal

Every module's config resolves through a stack of layers, key by key: the embedded template, a repo-wide default under the board dir, the worktree's own tracked file and an untracked local override under `.lyx/`.
A board task carries run settings, keys of any module's config, and batten writes them into the pair's local override, so a small task's settings need no hand step and never show up as a tracked config change at landing.

## Scope

In:

- Layered resolution in `internal/configengine` for per-worktree and hub-wide modules, with the per-key fallback down to the template.
- The repo default layer for per-worktree modules at `<BoardDir>/_lyx/config/<module>.yaml`, and the local override layer at `<anchor>/.lyx/config/<module>.local.yaml`.
- Reconcile across layers (`internal/configsync`, `internal/hubreconcile`), including the one-time migration of today's full per-worktree files to sparse overrides.
- `lyx config` layer selection and provenance display, and `lyx config reconcile` output per layer.
- A `run_settings` field on the board task, validated on write by `lyx board`, and batten writing it into the pair's local override files.
- The orch stencil's hand step for `review_max_bounces` replaced by run settings.
- Docs and PATTERN updates named under `## Constraints`.

Out:

- New config keys of any module, including the per-perch budget keys, which belong to `burler-split-budget`.
- Starting ready tasks mechanically (`orch-split`) and batten landing itself (`orch-wake-policy`).
- Any change to which modules are hub-wide.

## Decisions

### D1 — Layer stack and merge rule

Decision: a per-worktree module resolves as template, then repo default, then worktree file, then local override, each upper layer overriding the one below key by key.
A hub-wide module resolves as template, then hub file (`<BoardDir>/_lyx/config/<module>.yaml`, unchanged), then the reading worktree's local override, restricted by D4.
Mappings merge key by key at every depth.
Scalars and lists replace whole.
An open map (`configreg.Module.OpenMaps`) is carried whole from the highest layer that holds it, the rule `FillMissing` already applies between a file and its template.
Each top-level alias in `models` is carried whole the same way, matching the whole-entry replacement `modelspec.LoadRegistry` applies over its built-ins, so an upper layer's alias entry replaces the lower one outright and inherits none of its `defaults`.
No layer can delete a key from a layer below.
The layers are merged unexpanded, and the merged document is expanded once, through `envsource.Build` of the reading worktree's anchor for a per-worktree module and of the board dir for a hub-wide module; no layer is expanded against its own directory's `.env`.
Standalone, with no board dir, the stack is template, worktree file, local override.
Rationale: one per-key rule answers the board's first open question (yes, a key missing from a present file falls back to the layer below and finally to the template) and builds on the in-memory template fill both loaders have had since `5aa04c279`.

### D2 — Base layer is full, upper layers are sparse

Decision: the base layer is the lowest file layer: the repo default in a hub, the worktree file standalone.
Reconcile keeps the base layer full, adding template keys and pruning retired ones as today.
Every layer above it is sparse, holding only the keys it overrides: reconcile prunes its retired keys and runs the module's `Migrate` hook over it, and never adds a key.
`Set` and an editor edit of an absent sparse layer start from an empty mapping, not from the template.
A seed-only module's key set is the operator's at every layer: reconcile seeds its absent base layer once and never rewrites, prunes or migrates any of its layers, apart from D6's one-time strip.
Its layers merge by D1, so each top-level alias in `models` is carried whole from the highest layer holding it, and a lens string or fan list in `burler` replaces the lower one whole.
Of the seed-only modules, only `models` takes keys its template lacks in the local layer: D4 closes `burler`'s local layer.
Rationale: a full worktree file would shadow the repo default completely, making the default layer inert.
Pruning a seed-only layer against its template would delete the operator's own aliases, lenses and fans, which `configsync` protects today.

### D3 — File locations

Decision: the repo default of a per-worktree module is `<BoardDir>/_lyx/config/<module>.yaml`, beside the hub-wide modules' files, committed on Bolt and pushed through the same path as a hub-wide edit.
That directory now holds two kinds of file, told apart by `configreg.Module.HubWide`: a hub-wide module's hub file and a per-worktree module's repo default.
The local override is `<anchor>/.lyx/config/<module>.local.yaml`, in the never-tracked mirror of the config dir, for every module.
The `.local.yaml` suffix keeps it apart from the editor staging copy at `configengine.StagingFile`, so a staging file left by an aborted edit is never read as an override.
`internal/configengine` declares both accessors beside `ConfigFile`, and no other package forms either path.
Rationale: the operator named `_board` for the default and `.lyx/` for the local file.

### D4 — Local layer is bounded per module

Decision: `configreg.Module` gains `LocalKeys`, the keys of a hub-wide module a local override may set.
darn lists `writer`, `writer_timeout_min` and `verify_attempts`; landing lists `conflict`, `conflict_timeout_min`, `describe` and `describe_timeout_min`; board, fabric and gate list none.
darn keeps `verify` and `review_max_bounces` out, since `verify` runs a shell command and a `review_max_bounces` of 0 skips Darn-Review and Describe.
landing keeps `publish_verify` out because it runs a shell command, `require_pr_to_base` because a value without the parent branch makes Publish and PR-Gate return Done with no pull request or approval, and `squash` and `co_authored_by` because they shape the commit Finalize writes on the shared parent branch.
Gate lists none because its slots and `-p` cap are one hub-wide budget every pair shares, board none because its vocabulary is the one board's, and fabric none because its keys shape a pair at creation, before any local file exists.
`configreg.Module` also gains `LocalDenied`, the dotted key paths of a per-worktree module a local override may not set; every other key of a per-worktree module is open to it.
reed denies `shell`, `tmux`, `tmux_conf`, `mouse` and `debug_log`; shuttle denies `claude`, `run_dir` and every `claude_deny_*` key; loom denies `parent_review_wait_min`, `discussion_producer`, `discussion_fan` and `plan_fan`; burler denies `lenses` and `fans`; the remaining per-worktree modules deny none.
reed's and shuttle's keys are denied because each names an executable lyx runs, a file sourced into the hub's shared tmux server, an option the hub's shared tmux server takes at boot from whichever pair starts it, a directory whose sharing across worktrees makes the orphan sweep delete another worktree's run dirs, or a standing PreToolUse guard of every agent the pair spawns.
loom's `parent_review_wait_min` is denied because Discussion-Write's parent-review gate lets the discussion through once it elapses, so a value of 1 passes the parent's approval after one minute.
loom's `discussion_fan` and `plan_fan` are denied because each picks the lens set a review segment's forks are given, so a local empty value turns a hub-chosen fanned review back to a solo one, untracked.
loom's `discussion_producer` is denied because `twill` makes the Discussion-Write writer fork one reviewer per lens of burler's `twill-discussion` fan over its own record, so a local value of `single` or `seats` switches off a hub-chosen review, untracked.
burler's `lenses` and `fans` are denied because they are the review prose and the lens set every review fork is given, so a local value could hollow a review untracked; burler's local layer therefore holds no key.
The lists come from testing every key of every module, hub-wide and per-worktree, against these categories: runs an executable, sources a file, reaches across pairs, including a shared process's boot option or what lands on the parent branch, or switches off or hollows a guard, including a review or an approval gate, a timeout whose expiry lets that gate pass, and the prose or lens set a reviewer is given.
The test asks what a key's value can do to a guard, not what kind of key it is: a timeout or a seed-only key meets a category as any other key does.
A key the plan touches, and a key added later, takes the same test: one that meets a category joins its module's `LocalDenied` or, for a hub-wide module, stays out of `LocalKeys`.
A local file carrying a key outside its hub-wide module's `LocalKeys`, or inside its per-worktree module's `LocalDenied`, fails the load with `ErrInvalid` naming the file and the key; it is never silently ignored.
Bound: a local override can never change a key that runs an executable, sources a file into a shared process, sets a shared process's boot option, switches off or hollows an agent guard, a review or an approval gate, shapes what lands on the parent branch or is a fact shared across pairs; what it can change is session tuning, such as timeouts, budgets, model picks and colors.
What Seed-Child writes comes from the board entry, whose history shows who set it.
A later edit of the local file in the pair, through `lyx config --layer local` or an editor, is untracked: no history records it and landing does not show it (R3).
Rationale: hub-wide modules are hub-wide because a task branch must not change them; an untracked local file persists in the pair, reaches every later driver and spawn that loads the module and is writable by the pair's agents, so both lists keep the dangerous keys out of it and the local layer adds no untracked route to them.
The pair's git-ignored `.env`, which `envsource.Build` reads to expand `${env:...}` markers at load, is already such a route for every denied key that carries a marker; D4 leaves it as it is (R3).
This narrows the operator's "any key of any module" (see R1).

### D5 — Strictness across layers

Decision: `Load` and `LoadOrTemplate` keep their policies, applied to the base layer: strict `Load` errors when the base layer file is absent, and `LoadOrTemplate` degrades to the template.
An absent upper layer file is normal under both and adds nothing, and so is a present local file that is empty or holds only comments.
A present layer file that fails to parse returns `ErrInvalid` naming that file.
A local file naming a key its module's template lacks, or a key D4 closes to the local layer, returns `ErrInvalid` naming the file and the key.
Two kinds of key are exempt from the template-lacks refusal, matching D8's validation: a key path under any top-level alias in the `models` local file, whether or not the template holds that alias, because its aliases are the operator's (D2) and the models loader validates each alias block at load, and a key path under one of the module's open maps (`configreg.Module.OpenMaps`), whose entries are open-ended by design.
The base layer and the worktree file keep today's tolerance of keys the template lacks, which `lyx config --set` reports as `preserved` and reconcile prunes.
A filled key is logged with the layer it came from.
Rationale: sparse upper layers make absence the common case; strictness belongs to the layer whose absence means a broken hub, and the unknown-key refusal to the one layer a tool writes from the board.

### D6 — One-time migration to sparse files

Decision: the hub reconcile walk seeds each per-worktree module's absent repo default from the prime's file, as `SeedsFromPrime` does for a hub-wide module.
A tracked marker file in each worktree's `_lyx/config/` records that the worktree's files are sparse.
The walk strips, from each worktree lacking the marker, every key whose value equals what the layers below resolve to, deletes a file left empty, and writes the marker, all in that worktree's existing commit.
The comparison is over unexpanded YAML nodes: a `${env:...}` marker equals only the same marker text, so a literal equal to the walk's own expansion of a marker below is kept.
An open map is compared and stripped as one unit: it is kept whole unless it equals the resolved map below, because D1 carries it whole and a stripped entry would fall to the template.
A `models` alias is compared and stripped as one unit in the same way.
The strip never changes what a key the worktree file holds resolves to.
Seeding the default can change a key the worktree file lacks: a key the prime's file holds beyond the template, such as a seed-only alias, lens or fan, now resolves to the prime's value beneath that worktree.
The walk reports each such key as an `inherited` list beside `stripped`, per worktree and module, so the operator can pin the old value with `lyx config <module> --set` in that worktree.
The strip cannot tell a deliberate pin from a copied value: a worktree value equal to the seeded default becomes an inherited one, and a later `--layer default` change reaches it with no change in the worktree's own files.
Bound: the walk reports every stripped key, per worktree and module, as a `stripped` list on that layer's `lyx config reconcile` result, so the operator can restore a pin with `lyx config <module> --set` in that worktree.
A worktree carrying the marker is never stripped.
A pair forked from a migrated worktree inherits the marker with its files; one forked before is stripped by the next walk.
A walk that dies partway, or skips a pair as being created or mid-merge, leaves those worktrees without the marker, and the next walk strips them.
Rationale: today's full files would otherwise shadow the new default; tying the strip to a per-worktree marker makes the migration resumable, and never stripping a marked worktree keeps a later change to the default from erasing a deliberate override that equalled the old one.

### D7 — `lyx config` surface

Decision: `lyx config <module>` gains `--layer default|worktree|local`.
Without it, a per-worktree module edits its worktree file and a hub-wide module its hub file, as today.
`--layer default` writes `<BoardDir>/_lyx/config/<module>.yaml`, the repo default of a per-worktree module and the hub file of a hub-wide one, through the hub-wide write path: board write lock, strict validation with restore on failure, commit on Bolt and push, and the same refusal from a non-prime worktree or a slug-named session.
`--layer worktree` is refused for a hub-wide module.
`--layer local` writes the anchor's local file, uncommitted, and refuses a key D5 would refuse at load.
`--print` prints the selected layer's file verbatim; `--resolved` prints the merged result, each leaf annotated with the layer it came from (`template`, `default`, `worktree`, `local`).
`lyx config reconcile` reports one result per module and layer, with a `layer` field and, for the D6 migration, `stripped` and `inherited` lists.
`lyx config menu` marks each module with the layers that hold a file.
Rationale: answers the board's second open question; the default layer is a hub-level write and takes the hub-wide guard unchanged.

### D8 — `run_settings` on the board task

Decision: `boardengine.Task` gains `run_settings`, an object of module name to a mapping of that module's keys, nested as in the YAML file; dotted keys are not used.
Upsert and upsert-batch accept it; an empty object clears it.
`lyx board` runs each module's `Migrate` hook over its part, stores the migrated result, and validates it through `internal/configreg`: the module is registered, each key path exists in its template, sits under one of its open maps or is a key path under any `models` alias, a value's shape (mapping or not) matches the template's where the template holds the path, and the key is open to the local layer under D4.
A write that sets or changes `run_settings` is refused from a session whose strand name carries a slug, through the same guard function as the hub-wide config write, which parses the name with `internal/agentname`; it stops a mistaken agent write, not a determined one.
A write that changes `run_settings` on a task whose status shows a run past Seed-Child returns a notice that the change applies to the next run's Seed-Child, and that the running pair takes it through `lyx config <module> --layer local --set` in the pair.
`lyx board intake` never sets it.
Value semantics, such as a positive integer, are checked when the module loads, not at board write.
Rationale: the board stores tasks as JSON; one nested shape mirrors the files batten writes, and migrating then validating at write catches a typo or a retired key before a run starts.

### D9 — Batten writes the pair's local overrides at Seed-Child

Decision: batten's Seed-Child row reads the task's `run_settings` once, fresh from the board, runs each module's `Migrate` hook over its part, validates it as D8 does, and writes one `<pair>/.lyx/config/<module>.local.yaml` per named module, atomically, before it writes the child's seed.
It removes the local file of every registered module the run settings do not name, logging a Warn per removed file, so the pair's local layer is exactly the task's run settings at Seed-Child.
Worktree-Create refuses a pre-existing branch, so the only local files this can find are ones written into the pair while Seed-Child was Stuck, such as a hand-written `lyx config --layer local` override; the resume discards them, and an override placed through the D9 way forward, the board entry's `run_settings`, is the only one that survives it.
The read, migrate, validate, write and remove steps are injected `SeedChildDeps` closures wired in `internal/battencli` with the pair anchor handed in, as `ReadBoardType` is; `internal/battenshed`'s import allowlist is unchanged.
A key that does not validate routes Seed-Child to Stuck, naming the module, the key and the way forward: correct the task's `run_settings` with `lyx board upsert`, then `lyx batten run <slug>`.
A write or remove failure is a hard error.
Seed-Child is idempotent: a resumed Seed-Child redoes the whole write from the current board entry.
No other row and no child launch writes the files.
Bound: in a pair batten creates, batten writes the local override files once, at Seed-Child; afterwards they change only through `lyx config --layer local` or an editor.
Outside a batten pair the local layer is the operator's alone.
Rationale: Seed-Child already reads the board task fresh for `recipe` and runs before any child driver exists, so the write can neither race a live driver nor loop on Run-Shed's Stuck self-route.

### D10 — Orch stencil

Decision: `contracts/stencils/orch/orch-template-role.md`'s rule that a small task gets `review_max_bounces: 2` in its pair's loom config becomes: a small task gets `run_settings` `{"loom": {"review_max_bounces": 2}}` on its board entry before its batten run starts; never 1, as today.
Rationale: removes the hand step and the pair-exists check, which `orch-split` needs for a mechanical driver.

### Added after Discussion (operator, 2026-10-11): Landing-gating and review switch-off keys are not local

Decision: The local layer and run_settings may not set landing.require_pr_to_base, landing.squash, landing.co_authored_by or darn.review_max_bounces. A key that gates or shapes a landing into the parent branch, or switches off a review, is a refused category in D4's category test. Acceptance rows name landing.require_pr_to_base and a local darn.review_max_bounces: 0 among the refused keys.

Rationale: An untracked file the pair's agents can write, invisible at landing, would otherwise land unreviewed, unapproved code on the shared parent branch, against D4's own bound.

### Added after Discussion (parent, 2026-10-11): D4 rationale: the per-worktree .env is an untracked route

Decision: D4's rationale and R3's bound are corrected: the git-ignored per-worktree .env already binds the env markers of the denied per-worktree keys untracked and persistently in a pair. The deny lists stay, since the local layer must not add a second such route.

Rationale: configengine.load expands markers through envsource.Build(baseDir), which reads <anchor>/.env; the stated premise that those keys persist only through a tracked file is false.

### Added after Discussion (parent, 2026-10-11): D6 strip compares unexpanded values

Decision: D6's migration strip compares each value as written, before ${env:...} expansion, against the layer below's unexpanded value.

Rationale: An env-expanded comparison strips a literal that only equals this environment's expansion of a marker below, changing the worktree's resolution in any other environment.

### Added after Discussion (parent, 2026-10-11): Migration may add seeded-default keys

Decision: D6 and its acceptance criterion state that migration leaves every key a worktree's file held resolving unchanged, but may add keys the seeded default holds under a worktree whose file lacked them, such as a seed-only alias or lens the prime holds.

Rationale: The sparse layering resolves missing keys from the default, so the claim that every worktree's resolution is unchanged is false for keys the file lacked.

### Added after Discussion (parent, 2026-10-12): D4: loom.discussion_producer is LocalDenied; lists derived from the category test

Decision: loom.discussion_producer joins loom's LocalDenied: twill makes the Discussion-Write writer fork one reviewer per lens of burler's twill-discussion fan, so a local single or seats switches off a hub-chosen review. The plan derives LocalKeys and LocalDenied by applying D4's category test to every key of every module's template, recording per key the category it meets or none, rather than patching individual keys; a test pins that every template key is classified.

Rationale: Round 4's review tested every template key and found only this one misplaced; the recurring D4 finding came from enumerating the lists instead of deriving them, which a per-key derivation in Plan and a pinning test end.

### Added after Discussion (parent, 2026-10-12): D1: markers expand once, after the merge

Decision: Layers merge unexpanded; the merged document is expanded once through envsource.Build of the reading worktree's anchor for a per-worktree module and of the board dir for a hub-wide module. No layer is expanded against its own directory's .env. An acceptance row pins that a ${env:...} marker the worktree file inherits from the repo default expands against the reading worktree's .env.

Rationale: D6's claim that the strip never changes a held key's resolution holds only under single post-merge expansion; it matches today's one-expansion load and keeps per-worktree .env bindings such as LYX_SHUTTLE_CLAUDE working after migration.

### Added after Discussion (parent, 2026-10-12): D6: strip acceptance covers both marker directions

Decision: The strip's acceptance row also pins the reverse case: a ${env:...} marker in the worktree file over a literal in the repo default equal to that marker's expansion is kept, not stripped.

Rationale: D6 compares unexpanded values in both directions; the reverse case is the common one after migration and would otherwise silently drop a .env binding.

## Constraints

- `PATTERN-hub-wide-config` changes: `<BoardDir>/_lyx/config/` also holds per-worktree modules' repo defaults, and a hub-wide module also reads the reading worktree's local override, restricted to `LocalKeys`.
  The entry and its background are updated in the same commit.
- `PATTERN-config-strictness` changes: strictness applies to the base layer, the missing-key fill runs layer by layer down to the template, and a local file refuses unknown keys.
  The entry's background is updated in the same commit.
- `PATTERN-durable-vs-ephemeral-state` is narrowed: the local override sits in the mirrored directory, not at the mirrored file path, because the mirrored path is the editor staging copy's.
  The background says so in the same commit.
- New PATTERN entry `PATTERN-config-layers`: resolving a module's config, only `internal/configengine` reads or stacks layer files, and only it forms their paths; a key that runs an executable, sources a file, reaches across pairs, including a shared process's boot option or what lands on the parent branch, or switches off or hollows a guard, including a review or an approval gate, a timeout whose expiry lets that gate pass, and the prose or lens set a reviewer is given, is closed to the local layer through D4's lists.
- Local override files are never disposable, for `lyx-cleanup` or any other cleanup.
- `PATTERN-told-geometry`: `configengine`, `hubreconcile` and `battenshed` are handed the board dir and the anchor; none derives them.
- `PATTERN-binary-change-reconcile`: the layer reconcile and the D6 migration run only inside `hubreconcile.Ensure`'s walk and pair call.
  The `Migrate` runs over run settings in D8 and D9 are not a reconcile of hub config and stay outside it.
- `PATTERN-refusal-way-forward`: the D9 Stuck reason names its way forward and gets a row under the shed section of `contracts/specs/refusal-spec.md`, since Seed-Child is a shed producer, and a test.
- `PATTERN-shed-run-directory`: batten writes no seed field for run settings; the local files carry them.
- `PATTERN-test-speed`, `PATTERN-test-economy`: layer resolution is pinned at `configengine`'s surface in Tier 1 tests; hubreconcile and batten scenarios extend their existing integration tests.
- Docs in the same commit: `internal/configengine/doc.go`, `internal/configcli/doc.go` and the `lyx config` help, `internal/configsync`, `internal/hubreconcile/doc.go`, `internal/boardengine/doc.go` and `lyx board` help, `internal/battenshed/doc.go`, and `docs/overview.md` where it describes config resolution.

## Auto-mode assumptions

- D4 narrows the operator's "any key of any module" for hub-wide modules and for reed's, shuttle's, loom's and burler's denied keys.
- A running pair is not rewritten when its task's `run_settings` change (D8, D9).
- Run settings may sit on a note; only a task's run reads them.

## Open risks

- R1: D4 departs from an operator ruling on security grounds; parent review confirms or overrules it.
- R2: The D6 strip commits a config change in every running pair; `fabricengine.ReadConfigChanges` flags a pair's file at landing when it differs from both its fork point and the parent's current copy, so a pair whose overrides differ from the prime's sees one landing warning after the migration.
- R3: An agent in a pair can still write its own local file, and nothing at landing shows it; D4's lists bound what that file gains to session tuning.
  The pair's git-ignored `.env` stays an untracked route to every marker-carrying denied key, such as `LYX_SHUTTLE_CLAUDE` for `shuttle.claude`.
  `envsource.Build` lets the process environment override `.env`, so a `LYX_*` variable in the environment of the process that boots the hub's shared tmux server reaches every later pane of every pair; this cross-pair route belongs to the same parent decision (Q12).
  Closing either route is outside this task, and the parent decides whether they get an entry of their own.
- R4: The board's `kickstart_pack` incident is not explained by current code, which fills a missing key from the template; its cause stays unknown, and the parent decides whether it gets a note of its own.
- R5: `burler-split-budget` (running) adds per-perch loom keys; a run setting naming one validates only once that task lands, and the hub-level per-perch values it plans go into the repo default via `lyx config loom --layer default` if this task lands first.
- R6: `orch-wake-policy` (running) also changes batten and moves the post-landing config reconcile into batten; whichever lands second rebases its batten changes onto the other.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 4: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-4-review.md; fixer report: _lyx/reviews/discussion/round-4-fixer-report.md.
  - d1-env-expansion-across-layers-undecided (class decision, severity MEDIUM, open in rounds 4)
  - d4-bound-false-for-per-worktree-modules (class design, severity BLOCKING, open in rounds 1, 2, 3, 4)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A key absent from every file layer resolves to the template default, and a key present in several layers resolves to the highest one, at every mapping depth.
- A list or scalar in an upper layer replaces the lower one whole; an open map is taken whole from the highest layer holding it.
- Standalone, with no board dir, resolution uses template, worktree file and local file only.
- Strict `Load` errors on an absent base layer, and `LoadOrTemplate` degrades; an absent upper layer, and an empty or comment-only local file, is no error under either.
- A local file with an unparseable body, a key the template lacks, a hub-wide key outside `LocalKeys` (including `darn.verify`, `darn.review_max_bounces`, `landing.publish_verify` and `landing.require_pr_to_base`) or a per-worktree key in `LocalDenied` (including `reed.shell`, `reed.mouse`, `shuttle.claude_deny_python`, `loom.parent_review_wait_min`, `loom.discussion_producer`, `loom.plan_fan` and `burler.lenses`) fails the load naming that file; a key the template lacks in the base layer or the worktree file is tolerated as today.
- A `models` local file and `run_settings` accept an alias the template lacks, and an entry such as `opus.defaults.version` under an alias the template holds.
- A local `models` alias without a `defaults` block resolves with no defaults from the layers below.
- A local file and `run_settings` accept an entry under an open map that the template lacks, such as `shuttle.claude_prompt_cache_ttl_roles.<role>`, and the module loads.
- A stale editor staging file at `StagingFile` is never read as a local override.
- `Set` into an absent sparse layer writes only the set key.
- Reconcile adds missing keys only to the base layer, prunes retired keys and runs `Migrate` in every layer but a seed-only module's, whose present layers it leaves untouched, and reports each result with its layer.
- The migration walk seeds an absent repo default from the prime, strips keys equal to the layers below from each unmarked worktree, reports each stripped key, deletes an emptied file and writes the marker; a second walk strips nothing.
- After the strip every key a migrated worktree's file holds resolves to the same value as before, including an open map that differs from the default's in one entry a literal equal to the walk's expansion of a `${env:...}` marker below, which is kept, and a `${env:...}` marker over a literal in the repo default equal to the walk's expansion of that marker, which is kept.
- A `${env:...}` marker the worktree file inherits from the repo default expands against the reading worktree's `.env`.
- A seed-only key the prime's file holds and a worktree's file lacks is reported in that worktree's `inherited` list.
- A walk interrupted after the default is committed, or one that skipped a pair as being created or mid-merge, is completed by the next walk, which strips each worktree still lacking the marker.
- `lyx config --layer default` is refused from a non-prime worktree and from a slug-named session; `--layer worktree` is refused for a hub-wide module; `--resolved` names each leaf's layer.
- `lyx board upsert` migrates `run_settings` and refuses one naming an unknown module, a key absent from the template that is neither under a `models` alias nor under an open map, a shape mismatch, or a key D4 closes to the local layer, and refuses a `run_settings` change from a slug-named session; a change on a running task returns the D8 notice.
- Seed-Child writes exactly the named modules' local files and removes the others; with empty or absent `run_settings` it removes every local file and resolution falls through to the layers below.
- Seed-Child interrupted between its writes and removals and then resumed leaves the local layer equal to the board entry's current `run_settings`.
- A run-settings key that does not validate routes Seed-Child to Stuck with the way forward named.
- A pair whose `loom` run settings set `review_max_bounces: 2` runs its review segments with budget 2, and landing reports no per-worktree config change for it.

## Notes for the plan writer

- `yamlengine.FillMissing` already fills one document from another; stacking it from the top layer down is one route to D1.
- Loom-Preflight's retired-key refusal (named in `internal/hubreconcile/doc.go`'s accepted gaps) reads per-worktree files and needs to see every layer.
- `internal/loomcli/darnwiring.go` loads darn from the board dir alone; D4 needs the pair's anchor passed too.
- `configsync.ReconcileHubWideAt` and `ReconcileAll`'s `Retired` and `Divergent` handling assume `<BoardDir>/_lyx/config/` holds hub-wide modules only, and need the D3 distinction.
- Degrading callers that load early, such as `loggerconfig`, need the board dir handed in to read the repo default.
- The knob-validation comment in `internal/websterengine/config.go` still says a missing key gets Go's zero value, which the template fill no longer allows.
