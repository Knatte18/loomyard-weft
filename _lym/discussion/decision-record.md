# Decision record — config keys nest under the area they govern, named with dots

## Goal

A config key's name shows the area it governs, one dot per level, with its file first:
`loom.discussion.review_ask_parent`, `loom.code.max_bounces`, `drift.judge_ask_parent`.
`loom.yaml` and `tapestry.yaml` stop spelling the area into flat key names and hold one map per area.
Every hub's existing values, in every layer and in the board's `run_settings`, carry over to the new names with no operator step.
Every message, template comment, doc, stencil, spec and recipe names a key by its full dotted name.

## Scope

In:

- The key layout of `internal/loomengine/template.yaml` and `internal/loomengine/tapestry.yaml`, and every reader of those keys.
- A move table that carries old key names to new paths in every config file layer, in `profiles.yaml` and in the board entries' `run_settings`.
- Refusal, with the new name as the way forward, of an old name written after the move.
- Full dotted key names in every Go message, every module's template comments, `contracts/` stencils, specs and recipes, `pattern/` and `docs/`.

Out:

- Nesting the keys of any other template (D2).
- The shape of a board entry's `run_settings`: it stays an object of module name to that module's keys, nested as the module's file is.
- A new `lym config` form: `lym config <module> --set <dotted path>=<value>` already takes a dotted path.
- Historical measurement records under `docs/research/`, which keep the names of their run.
- Whether `loom.code.auto_accept`'s narrowed meaning needs a rewrite rather than a flag (note `code-auto-accept-flag-only`).

## Decisions

### D1 — loom.yaml and tapestry.yaml nest one level per area

Each prefixed key moves under a map named for its area, keeping the rest of its name.
A bare key that is an agent's model-spec and names an area becomes that area's `model`.

| Old key | New path |
|---|---|
| `discussion` | `discussion.model` |
| `discussion_<rest>` (all 18) | `discussion.<rest>` |
| `code_<rest>` (all 13) | `code.<rest>` |
| `split_<rest>` (all 8) | `split.<rest>` |
| `raddle` | `raddle.model` |
| `raddle_auto_approve` | `raddle.auto_approve` |
| `friction` | `friction.model` |
| `friction_timeout_min` | `friction.timeout_min` |
| `pr_review_<rest>` | `pr.review_<rest>` |
| `review_<rest>` (all 6) | `review.<rest>` |
| `reviewer_model`, `fixer_model`, `judge`, `seed`, `assist_reviewer_timeout_min` | `review.reviewer_model`, `review.fixer_model`, `review.judge`, `review.seed`, `review.assist_reviewer_timeout_min` |
| tapestry `plan` | `plan.model` |
| tapestry `plan_<rest>` | `plan.<rest>` |
| tapestry `parts` | `parts.model` |
| tapestry `parts_<rest>` | `parts.<rest>` |
| tapestry `part_writer` | `parts.writer` |

`loom.yaml`'s `subagent_model`, `driver`, `step_idle_timeout_min` and `notify_parent_on` and `tapestry.yaml`'s `implementer` stay top-level.
Every key keeps its value, default, comment meaning and category.

Rationale: the operator's two examples, `discussion.review_ask_parent` and `code.max_bounces`, fix the rule as "area, then the rest of the name".
The run-wide review keys join `review` because a bare `loom.judge` reads like drift's judge, the confusion the entry names.
`tapestry.yaml` must follow, because the per-perch budget and model keys are built from the perch name for the plan perch the same way as for the loom perches.

### D2 — No other template nests

Every other module keeps its keys as they are, drift included.

Rationale: the operator's own example `drift.judge_ask_parent` keeps drift's key flat, and no other file shares loom's perch key machinery.

### D3 — A key is named in full everywhere

Every Go message, refusal, way forward and halt notice that names a config key renders `<module>.<dotted path>`, for every module's keys.
Go renders the name through one helper, `configengine.KeyName` (new), from the module's name constant and the key's path constant.
Each key path a message or reader cites is one named constant in its module.
Every module's template comments, the stencils, specs and recipes under `contracts/`, `pattern/` and `docs/` outside `docs/research/` name a key the same way.
A command a way forward prints keeps the CLI's own form, `lym config loom --set code.max_bounces=2`.

Rationale: the entry asks that a reader can tell from the name alone which file and area a key governs.
One helper and one constant per key keep a rename from leaving a stale name in a message, as `PATTERN-code-conventions` asks.

### D4 — A module's move table names every moved key, and an old name is refused after the move

`configengine.Spec` gains `MovedKeys` (new), mapping each old dotted key to its new dotted path; loom's and tapestry's specs fill it from D1's table.
An old name is caught by its own check, separate from `RetiredKeys`:

- it matches a unit whose path equals the old key exactly, never a path below it;
- an old key that names its new area, such as `discussion`, matches only a non-mapping value, so every new `discussion.*` path is judged by the template and its category as any other key;
- it refuses whatever the old key's category was.

The check refuses an old name, naming the old and new full names and one way forward for its site:

- in a local or worktree file the loader reads, `lym config reconcile --apply` in that worktree, which moves it (D5);
- in a hub base file the loader reads, `lym config reconcile --layer default --apply` from the hub's prime worktree, which moves it (D5);
- in a task's `run_settings` and a profile block, through `configreg.PrepareRunSettings`, whose refusal names the old and new full names alone, so its caller's own way forward is the one printed:
  Seed-Child's existing wrapper names `lym board upsert` for the task and `lym config profiles` from the hub's prime worktree for the profile,
  and `lym board upsert` and the profile check refuse the write itself, which the operator repeats with the new name;
- in `lym config <module> --set`, the same command with the new path, such as `lym config loom --set code.max_bounces=2`;
- in a recipe's setting reference, such as `loom.discussion_reviewer_fan`, at recipe build, the reference with the new full name, such as `loom.discussion.reviewer_fan`.

In a worktree or hub base file a key the template lacks otherwise only warns, and the two reconcile commands are the ways forward that warning already names;
in a local file the check replaces the refusal of a key the template lacks for the old names, so that refusal names `lym config reconcile --apply`.

`lym config <module> --unset` admits an old name and removes it, so the operator's removal route stays open.
The loom spec's existing refusal of the tapestry plan keys keeps its old names and names the new `tapestry.plan.*` path.
Bound: the check refuses only names the table holds, and admits nothing the template and categories would refuse; a key no table names is judged as today.

Rationale: a tuned value is never lost without a signal, and `PATTERN-refusal-way-forward` wants the refusal to name its way forward.

### D5 — Every reconcile moves old keys in the files it reconciles

`configsync` applies the module's move table to every file it reconciles: a hub base file, a worktree file and a local file, before it merges, prunes or strips the file.
When a hub base file is absent and `configsync` seeds it from the prime worktree's copy, it moves that copy's old keys before `sparseSeed` prunes it to the template's units, so the seed carries their values.
It is idempotent: a file with no old key comes back unchanged, so no marker guards it, and each machine's walk carries that machine's own untracked local files.
The move keeps the value and its comment and every other key's comment.
An old bare key that names its new area is told from the new map by shape: a non-mapping value is the old key.
An old key whose value is null is dropped and reported, since the new path's absence reads the same default.
A file holding both an old key and its new path keeps the new path's value, drops the old key and reports the drop with both names.
`configsync`'s `Result` reports each move; the walk commits a moved worktree file as it commits any other write it makes there.
`hubreconcile`'s walk mails the hub's orch one line per walk that moved anything, naming each file and each old and new full name.
Every lym verb outside `lym config` that loads the loom or tapestry config runs `hubreconcile.EnsureOrWarn` before that load, as the start verbs do today, so the first such verb on a new build runs the walk.
Once the stamp names the running build, the call costs the stamp read and board file checks a start verb pays today.
The `lym config` verbs run no walk: they are D4's ways forward, and `lym config reconcile --apply` reconciles its own layer.
`EnsureOrWarn`'s warning names re-running the verb once the walk ends, beside the start verbs it names today.
Bound: the move only renames keys the table names; it never removes a key the table lacks and never touches the machine layer, which holds no loom or tapestry key.
Bound: the walk runs once per build per machine, at the first verb that loads the loom or tapestry config.
A file the walk has not moved keeps its old key until the next walk or the `lym config reconcile --apply` of its layer that D4's refusal names meanwhile, and that verb's load refuses it under D4.
That covers a file that gains an old key after the walk, by a merge, a pull or a pair the walk skipped as incomplete.
On a new build it also covers a file of a worktree the walk skipped as mid-merge, and any file when `EnsureOrWarn` failed or waited out `DefaultLockWait` behind another verb's walk;
re-running the verb once that walk ends loads the moved file.
Bound: while a worktree stays mid-merge or the board pull skips the hub-wide write, the stamp stays stale, so every such verb on the hub reruns the walk, taking the hub lock and pulling the board, until the merge ends or the board is brought up to date.

Rationale: the walk visits every file layer of the machine it runs on, and running it before the first load on a new build lets one rule cover the shared hub base, the tracked worktree files and each machine's local files, and lets in-flight runs carry over with no operator step.
Moving before the prune keeps a local file's old key from being pruned as unknown.

### D6 — configsync moves profiles.yaml's blocks before it follows the template

Reconciling `profiles.yaml`, `configsync` moves the loom and tapestry keys in every recipe block of every profile, using each module's move table,
in the file and in the template base it reads, before `yamlengine.FollowTemplate` merges the new template in.
The follow then compares file, base and template at the new paths:
a shipped block the operator never changed takes the new template, and a shipped value the operator changed stays theirs.
It moves through the D7 helper under D5's rule for a block holding both an old key and its new path.
It keeps the file's comments and every other key, and reports each move in `configsync`'s `Result`, which the walk mails with D5's lines.
Like D5 it is idempotent and needs no marker, and `lym config reconcile --layer default --apply` runs it too.

Rationale: a move after the follow would find an operator's changed value at the old key beside the template's default at the new path, and the template base the follow rewrites no longer tells the two apart; moving file and base first leaves that call to the follow.
`profiles.yaml`'s blocks hold other modules' keys, which D5's per-module move does not reach, so the profiles module moves them itself.

### D7 — Board run_settings are carried by every hub-wide walk

Each hub-wide walk, after its hub-wide config write has committed and outside it, moves the loom and tapestry keys of every board entry's `run_settings` to their new paths.
A `configreg` helper applies each module's move table to a module-keyed settings object, and D6 uses the same helper.
The rewrite goes through `boardengine`'s own write path under its lock, keeps every key it does not move, and covers every entry.
It is idempotent and needs no marker: a board with no old key is not written.
A failed read or write is logged, leaves the board as it was and marks the walk skipped, so the build stamp stays stale and the next walk retries, as after a skipped hub-wide write.
A walk whose hub-wide write was skipped runs no D7 carry; its stale stamp brings the retry.
The orch gets one line naming each entry and each old and new full name.
Bound: a walk runs once per build per machine, so an old-name entry written after it by an older build waits for the next build's walk; Seed-Child refuses it meanwhile under D4, naming the new name and `lym board upsert`.

Rationale: an entry not yet started would otherwise reach Seed-Child with an old name and be refused there under D4.
Running outside the hub-wide write keeps the two board writes from nesting.

### D8 — Every reader and writer uses the nested path

- `loomengine`'s config decodes each area as a nested struct.
- `reviewbudget`'s key builders return `<perch>.max_bounces`, `<perch>.hard_max_bounces` and `<perch>.auto_accept`, and the run-wide keys are `review.max_bounces`, `review.hard_max_bounces` and `review.amend_max_bounces`.
- Code that builds a `run_settings` or profile object from key paths, such as `lym beam merge-parent`'s budget settings, nests each dotted path into the object through one helper, so the object carries `code: {max_bounces: N}`, never a literal `code.max_bounces` key.
- The per-segment model, fan, judge and seed resolution reads `<segment>.<key>` and falls back to `review.<key>`.
- A recipe setting reference splits at its first dot into module and dotted path; its syntax check admits a dotted path, and the setting lookup reads the nested path.
- `loomengine.FlagCodeAutoAccept` reads `code.auto_accept`, since every reinterpretation runs after `configsync`'s move.

Rationale: the move is a rename only; every fallback and default keeps its meaning.

### D9 — Classification follows the paths

`ConfigSpec().Categories`, the open maps and the derived `LocalDenied` of loom and tapestry name the new dotted paths, each with its old key's category.

Rationale: `configengine` already classifies, layers, merges, sets and validates by dotted path, so only the names change.

### Added after Discussion (parent, 2026-11-23): F4: dotted names are rendered, input forms stay

Decision: The dotted full name, such as loom.discussion.review_ask_parent, is how messages, comments, docs, stencils and halt notices render a key. run_settings stays a module-keyed nested object, and --set keeps its form lym config loom --set discussion.review_ask_parent=true. No one-token --set form and no flat dotted run_settings keys.

Rationale: This is the decision record as it stands (Scope Out, support-log Q10); a one-token input form would widen the task.

### Added after Discussion (parent, 2026-11-26): Fix the plan defects round 3 listed but did not raise

Decision: Build the six defects listed under 'Real defects not raised' in _lym/reviews/plan/round-3-review.md as fixes, not as open risks: (1) card 9's stale-stamp test drives a verb whose nearest pre-run is loom's own and that loads the seeded key, or sets cobra.EnableTraverseRunHooks without t.Parallel and drives circling continue; (2) cards 2 and 5: the --unset removal route works for a file holding several old keys, one command removing them all, and a TestDispatchSet row holds two old keys; (3) card 8: replace the second-walk assertion that cannot fail with a tier0 row that a board with no old key and a directory at a rendered output path is not reported skipped; (4) card 9: the CodeReviewCaps row sets a hard cap beside code_max_bounces so a hub base holding only code_max_bounces 7 is not refused; (5) card 6: a walk that fails after the hub-wide commit still mails its move lines; (6) card 11: PATTERN-leaf-packages wins in internal/shellguard: pythonRefusal names the key's new full name as a literal and shellguard imports no configengine, a declared exception to PATTERN-config-key-names.

Rationale: The reviewer confirmed each against the code and rated (1) and (2) BLOCKING in a full round; a leaf package's import rule is structural, while a key name literal is checked by the rename's own search.

## Constraints

- `PATTERN-config-layers`: layers are read only through `configengine`, a reconcile on a binary change only through `hubreconcile.Ensure`, and no reconcile touches the machine layer.
- `PATTERN-tolerant-records`: D7's board rewrite keeps every key it does not move.
- `PATTERN-refusal-way-forward`: every D4 refusal names the new full name, and a test reaches it.
- `PATTERN-code-conventions`: a key path a message or reader cites is one named constant (D3).
- Fact, from the code at this branch's base: `configengine` already handles nested keys.
  `configengine.Spec`'s `Categories` are dotted paths, `configengine.ValidateLocal` walks leaf paths, and layer merging, `--set` and `--unset` take dotted paths.
  `configengine.TopLevelValue` reads top-level keys only, and D2 leaves every key its readers read flat.
- Fact, from `internal/hubreconcile/walk.go`: the hub-wide walk runs `configsync`'s reconcile, over every registered module's board file, before it applies reinterpretations, inside one hub-wide write; each worktree's reconcile commits what it wrote.
  D5 and D6 move inside `configsync`, and D8's reinterpretation read relies on that order.

## Auto-mode assumptions

- Run-wide review keys nest under `review` (D1).
- `pr_review_*` nests under `pr` (D1).
- A bare area model key becomes `<area>.model` (D1).
- The entry's "by a reinterpretation, as other key moves were" is read as "automatically, on the reconcile, as earlier moves were": `configsync`'s move table carries the file layers and `profiles.yaml`'s blocks, and each walk carries board `run_settings` (D5 to D7).
- An old name is refused after the move, not read as an alias (D4).

## Open risks

- The entry says `run_settings` and `lym config --set` "use the dotted full name, file included"; this record keeps `run_settings` as a nested module object and `lym config loom --set discussion.review_ask_parent=…`, with no form that takes `loom.discussion.review_ask_parent` as one token. The parent decides whether a one-token form is wanted.
- A hub whose machines run different builds: an older build loads a moved hub base file against its flat template.
  It refuses one holding a map where its old key was a scalar, `discussion`, `raddle` or `friction` in `loom.yaml` and `plan` or `parts` in `tapestry.yaml`,
  and otherwise warns of the moved maps and runs on template defaults for every moved key.
  It also prunes the new nested keys from its own machine's local files and still accepts old names in `run_settings` and profiles, until it upgrades.
- Any run that lands a new loom or tapestry key before this run merges lands it flat; the D1 template criterion fails at this run's merge until the key is nested.

## Acceptance criteria

- D1: the loom and tapestry templates hold every key of D1's table at its new path, with no top-level key starting with an area prefix the table moves, and the loaded config holds the template's defaults at those paths.
- D1, D9: a hub base `loom.yaml` setting `discussion.review_ask_parent: true` loads that value; a local file setting `discussion.advisors` is refused as before, by its new path.
- Review: D2 — no other module's template changes its keys.
- D3: a run-settings refusal, a local-layer refusal and a budget refusal name the key as `loom.<area>.<key>`, rendered through `configengine.KeyName` (new).
- Review: D3 — every template comment, stencil, spec, recipe, pattern and doc outside `docs/research/` names a config key by its full dotted name.
- D4: an old key in a local file, a worktree file, a hub base file, a task's `run_settings` at Seed-Child, a profile block, `lym config loom --set` and a recipe setting reference is each refused, naming the old and new full names and that site's way forward of D4, and Seed-Child's refusal prints one way forward.
- D4: a local file setting `discussion: {reviewer_fan: ""}` is refused as a Guard key by its new path, and one setting `discussion: {timeout_min: 60}` is admitted.
- D4: `lym config loom --unset code_max_bounces` removes the old key from a file that still holds it.
- D4: the loom spec still refuses `plan_max_bounces` and names `tapestry.plan.max_bounces`.
- D5: reconciling a hub base, a worktree and a local `loom.yaml` that set `code_max_bounces: 2` and `discussion: sonnet[medium]` leaves `code.max_bounces: 2` and `discussion.model: sonnet[medium]`, keeps the other keys' comments, reports the moves, and a second reconcile changes nothing.
- D5: a local file holding only old keys keeps their values through a reconcile that prunes unknown keys.
- D5: an absent hub base `loom.yaml` seeded from a prime copy holding `code_max_bounces: 2` holds `code.max_bounces: 2`.
- D5: a file setting both `code_max_bounces` and `code.max_bounces` keeps the new path's value and reports the dropped old key.
- D5: the walk mails the orch one line naming each moved key's old and new full names, and a walk that moved nothing mails none.
- D5: after a rebuild that leaves the build stamp stale, a non-start loom verb of an in-flight run whose hub base and local `loom.yaml` hold `code_max_bounces: 2` resolves `loom.code.max_bounces` to 2 and does not halt.
- D6: after the walk, an untouched seeded `profiles.yaml` parses to the same YAML value as the new template, and a profile the operator added with `loom: {code_max_bounces: 3}` holds `loom: {code: {max_bounces: 3}}`.
- D6: a seeded `profiles.yaml` whose shipped `solid.tapestry` block the operator changed to `loom` `code_max_bounces: 3` holds `loom` `code.max_bounces: 3` after the walk.
- D7: after the walk, a board entry with `run_settings` `{"loom": {"code_max_bounces": 2}, "drift": {"scope": "full"}}` holds `{"loom": {"code": {"max_bounces": 2}}, "drift": {"scope": "full"}}`, and a second walk writes no board commit.
- D7: a walk whose board write fails leaves the entry as it was and the build stamp stale, and the next walk carries it.
- D8: each perch's budget resolves from `<perch>.max_bounces`, falling back to `review.max_bounces`, for the discussion, code, split and plan perches.
- D8: the task `lym beam merge-parent` files carries `{"loom": {"code": {"max_bounces": N, "hard_max_bounces": M}}}` and its Seed-Child applies those budgets.
- D8: a segment with an empty `code.reviewer_model` takes `review.reviewer_model`, and a recipe reference `loom.split.reviewer_fan` passes recipe build and resolves the nested value.
- D8: `loomengine.FlagCodeAutoAccept` leaves a hub whose moved file sets `code.auto_accept: false` unflagged.

## PATTERN changes

- `PATTERN-config-layers` — changes: Loading, writing or changing config: layers only through `configengine`; a hub base file holds only tuned units; a meaning change ships a reinterpretation, a key move a move table, both run by `hubreconcile`; a per-task beam setting is read only via `dobbyengine`'s one lookup.
- `PATTERN-config-key-names` — adds: Naming a config key in a message, comment, doc or stencil: in full, `<module>.<dotted path>`, rendered in Go by `configengine`'s one key-name helper from named constants.

## Notes for the plan writer

- `PATTERN-config-layers`'s background needs the move table under "Keys the template lacks": a key its module's move table names is moved by every reconcile, refused at load with that layer's reconcile command, `lym config reconcile --apply` or `lym config reconcile --layer default --apply`, as the way forward, and removable by `--unset`.
- The template comments cross-reference keys through a `<segment>_` pattern, so they need a prose rewrite, not a find and replace.
- D5's `EnsureOrWarn` before every loom or tapestry config load widens `loomcli`'s `arm`, which runs it today for `start` and `resume` alone, and reaches each other CLI outside `lym config` that loads either config, including the loaders `webstercli` and `shuttleparity` reach.
- D5's re-run way forward changes `hubreconcile`'s `reconcileWayForward`, which names start verbs alone today.
