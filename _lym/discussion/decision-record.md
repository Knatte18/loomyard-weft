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

The check refuses an old name, naming the new full name and `lym config reconcile`, which moves it, as the way forward:

- in a local file, a worktree file or a hub base file the loader reads, where a key the template lacks otherwise only warns;
- in a task's `run_settings` and a profile block, through `configreg.PrepareRunSettings`;
- in `lym config <module> --set`;
- in a recipe's setting reference, such as `loom.discussion_reviewer_fan`, at recipe build.

`lym config <module> --unset` admits an old name and removes it, so the operator's removal route stays open.
The loom spec's existing refusal of the tapestry plan keys keeps its old names and names the new `tapestry.plan.*` path.
Bound: the check refuses only names the table holds, and admits nothing the template and categories would refuse; a key no table names is judged as today.

Rationale: a tuned value is never lost without a signal, and `PATTERN-refusal-way-forward` wants the refusal to name its way forward.

### D5 — Every reconcile moves old keys in the files it reconciles

`configsync` applies the module's move table to every file it reconciles: a hub base file, a worktree file and a local file, before it merges, prunes or strips the file.
It is idempotent: a file with no old key comes back unchanged, so no marker guards it, and each machine's walk carries that machine's own untracked local files.
The move keeps the value and its comment and every other key's comment.
An old bare key that names its new area is told from the new map by shape: a non-mapping value is the old key.
An old key whose value is null is dropped and reported, since the new path's absence reads the same default.
A file holding both an old key and its new path keeps the new path's value, drops the old key and reports the drop with both names.
`configsync`'s `Result` reports each move; the walk commits a moved worktree file as it commits any other write it makes there.
`hubreconcile`'s walk mails the hub's orch one line per walk that moved anything, naming each file and each old and new full name.
Bound: the move only renames keys the table names; it never removes a key the table lacks and never touches the machine layer, which holds no loom or tapestry key.
Bound: the walk runs once per build per machine, so a file that gains an old key after it, by a merge, a pull or a pair the walk skipped as incomplete, keeps the key until `lym config reconcile` or the next build's walk; D4's refusal names that way forward meanwhile.

Rationale: the reconcile already visits every file layer, on every machine, before any command runs, so one rule covers the shared hub base, the tracked worktree files and each machine's local files.
Moving before the prune keeps a local file's old key from being pruned as unknown.

### D6 — profiles.yaml is carried once per hub by a reinterpretation

A `configreg.Reinterpretation` on the profiles module moves the loom and tapestry keys in every recipe block of every profile in the hub's `profiles.yaml`, using each module's move table.
It runs in the walk after `configsync`'s merge, so a profile the operator never changed has already taken the new template.
It keeps the file's comments and every other key, and its lines go to the orch with the walk's other reinterpretation lines.

Rationale: `profiles.yaml` is a seed-only file whose blocks hold other modules' keys, which `configsync`'s per-module move cannot see; the board marker of a reinterpretation runs it once per hub.

### D7 — Board run_settings are carried by every hub-wide walk

Each hub-wide walk, after its hub-wide config write has committed and outside it, moves the loom and tapestry keys of every board entry's `run_settings` to their new paths.
A `configreg` helper applies each module's move table to a module-keyed settings object, and D6 uses the same helper.
The rewrite goes through `boardengine`'s own write path under its lock, keeps every key it does not move, and covers every entry.
It is idempotent and needs no marker: a board with no old key is not written, and a failed read or write is logged and retried by the next walk.
The orch gets one line naming each entry and each old and new full name.
Bound: a walk runs once per build per machine, so an old-name entry written after it by an older build waits for the next walk; Seed-Child refuses it meanwhile under D4, naming the new name.

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

## Constraints

- `PATTERN-config-layers`: layers are read only through `configengine`, a reconcile on a binary change only through `hubreconcile.Ensure`, and no reconcile touches the machine layer.
- `PATTERN-tolerant-records`: D7's board rewrite keeps every key it does not move.
- `PATTERN-refusal-way-forward`: every D4 refusal names the new full name, and a test reaches it.
- `PATTERN-code-conventions`: a key path a message or reader cites is one named constant (D3).
- Fact, from the code at this branch's base: `configengine` already handles nested keys.
  `configengine.Spec`'s `Categories` are dotted paths, `configengine.ValidateLocal` walks leaf paths, and layer merging, `--set` and `--unset` take dotted paths.
  `configengine.TopLevelValue` reads top-level keys only, and D2 leaves every key its readers read flat.
- Fact, from `internal/hubreconcile/walk.go`: the hub-wide walk runs `configsync`'s reconcile, over every registered module's board file, before it applies reinterpretations, inside one hub-wide write; each worktree's reconcile commits what it wrote.
  D5, D6 and D8's reinterpretation read rely on that order.

## Auto-mode assumptions

- Run-wide review keys nest under `review` (D1).
- `pr_review_*` nests under `pr` (D1).
- A bare area model key becomes `<area>.model` (D1).
- The entry's "by a reinterpretation, as other key moves were" is read as "automatically, on the reconcile, as earlier moves were": `configsync`'s move table carries the file layers, a reinterpretation carries `profiles.yaml`, and each walk carries board `run_settings` (D5 to D7).
- An old name is refused after the move, not read as an alias (D4).

## Open risks

- The entry says `run_settings` and `lym config --set` "use the dotted full name, file included"; this record keeps `run_settings` as a nested module object and `lym config loom --set discussion.review_ask_parent=…`, with no form that takes `loom.discussion.review_ask_parent` as one token. The parent decides whether a one-token form is wanted.
- A hub whose machines run different builds: an older build loads the moved hub base `loom.yaml` against its flat template and refuses it, prunes the new nested keys from its own machine's local files, and still accepts old names in `run_settings` and profiles, until it upgrades.
- Any run that lands a new loom or tapestry key before this run merges lands it flat; the D1 template criterion fails at this run's merge until the key is nested.

## Acceptance criteria

- D1: the loom and tapestry templates hold every key of D1's table at its new path, with no top-level key starting with an area prefix the table moves, and the loaded config holds the template's defaults at those paths.
- D1, D9: a hub base `loom.yaml` setting `discussion.review_ask_parent: true` loads that value; a local file setting `discussion.advisors` is refused as before, by its new path.
- Review: D2 — no other module's template changes its keys.
- D3: a run-settings refusal, a local-layer refusal and a budget refusal name the key as `loom.<area>.<key>`, rendered through `configengine.KeyName` (new).
- Review: D3 — every template comment, stencil, spec, recipe, pattern and doc outside `docs/research/` names a config key by its full dotted name.
- D4: an old key in a task's `run_settings`, a profile block, `lym config loom --set`, a local file, a worktree file and a recipe setting reference is each refused, naming the new full name and `lym config reconcile` as the way forward.
- D4: a local file setting `discussion: {reviewer_fan: ""}` is refused as a Guard key by its new path, and one setting `discussion: {timeout_min: 60}` is admitted.
- D4: `lym config loom --unset code_max_bounces` removes the old key from a file that still holds it.
- D4: the loom spec still refuses `plan_max_bounces` and names `tapestry.plan.max_bounces`.
- D5: reconciling a hub base, a worktree and a local `loom.yaml` that set `code_max_bounces: 2` and `discussion: sonnet[medium]` leaves `code.max_bounces: 2` and `discussion.model: sonnet[medium]`, keeps the other keys' comments, reports the moves, and a second reconcile changes nothing.
- D5: a local file holding only old keys keeps their values through a reconcile that prunes unknown keys.
- D5: a file setting both `code_max_bounces` and `code.max_bounces` keeps the new path's value and reports the dropped old key.
- D5: the walk mails the orch one line naming each moved key's old and new full names, and a walk that moved nothing mails none.
- D6: after the walk, an untouched seeded `profiles.yaml` equals the new template, and a profile the operator added with `loom: {code_max_bounces: 3}` holds `loom: {code: {max_bounces: 3}}`.
- D7: after the walk, a board entry with `run_settings` `{"loom": {"code_max_bounces": 2}, "drift": {"scope": "full"}}` holds `{"loom": {"code": {"max_bounces": 2}}, "drift": {"scope": "full"}}`, and a second walk writes no board commit.
- D7: a walk whose board write fails leaves the entry as it was, and the next walk carries it.
- D8: each perch's budget resolves from `<perch>.max_bounces`, falling back to `review.max_bounces`, for the discussion, code, split and plan perches.
- D8: the task `lym beam merge-parent` files carries `{"loom": {"code": {"max_bounces": N, "hard_max_bounces": M}}}` and its Seed-Child applies those budgets.
- D8: a segment with an empty `code.reviewer_model` takes `review.reviewer_model`, and a recipe reference `loom.split.reviewer_fan` passes recipe build and resolves the nested value.
- D8: `loomengine.FlagCodeAutoAccept` leaves a hub whose moved file sets `code.auto_accept: false` unflagged.

## PATTERN changes

- `PATTERN-config-layers` — changes: Loading, writing or changing config: layers only through `configengine`; a hub base file holds only tuned units; a meaning change ships a reinterpretation, a key move a move table, both run by `hubreconcile`; a per-task beam setting is read only via `dobbyengine`'s one lookup.
- `PATTERN-config-key-names` — adds: Naming a config key in a message, comment, doc or stencil: in full, `<module>.<dotted path>`, rendered in Go by `configengine`'s one key-name helper from named constants.

## Notes for the plan writer

- `PATTERN-config-layers`'s background needs the move table under "Keys the template lacks": a key its module's move table names is moved by every reconcile, refused at load with `lym config reconcile` as the way forward, and removable by `--unset`.
- The template comments cross-reference keys through a `<segment>_` pattern, so they need a prose rewrite, not a find and replace.
