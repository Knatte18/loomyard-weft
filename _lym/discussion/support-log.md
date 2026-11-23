# Support log — config keys nest under the area they govern

## Interview

Autonomous run: no operator answers.
Each turn below is an exploration result or a self-made pick; the picks are in the question ledger.

### Turn 1 — the board

The entry asks for nested `loom.yaml` keys named with dots, carried over by a reinterpretation, and a choice of which other templates join.
No `Decision for config-keys-nested-by-area:` note exists.
Note `code-auto-accept-flag-only` touches `code_auto_accept`'s reinterpretation; it stays out of scope, and D8 keeps its reinterpretation reading the moved key.

### Turn 2 — exploration

Four subagents reported:

- `configengine` already handles dotted categories, leaf-path validation, deep layer merging and dotted `--set` and `--unset`; flat assumptions sit in `loomengine`'s struct tags, `reviewbudget`'s key builders, the open-map lists and `TopLevelValue`.
- `configreg.Reinterpretation` rewrites only the hub base file; worktree and local files can only be flagged by it; nothing rewrites board `run_settings` today.
  Earlier key moves ran through `configsync`'s move step, deleted in 3d3b48127.
- `loom.yaml` holds 86 keys in eight prefix groups; `tapestry.yaml` holds the `plan_*` and `parts_*` keys; drift's `judge_*`, gateslot's `redundancy_*` and landing's `finalize_fix_*` are the other groups.
- Messages spell keys inline in several styles; recipes already use `loom.<key>` setting references.

### Turn 3 — reading the walk

`internal/hubreconcile/walk.go` runs `configsync` before reinterpretations, and `configsync.ReconcileAll` covers worktree and local files, which the walk commits.
That settled D5: a move table in `configsync` carries every file layer on every machine, idempotently.

### Turn 4 — self-picks

Made picks Q1 to Q8 in the question ledger and wrote the record.

### Turn 5 — lens review

Every fork returned a report.

- walkthrough, bound-breaker, interactions — D4's refusal derived from `RetiredKeys` covers the new area paths by prefix and admits `none` keys: fixed; D4 is its own exact-path, shape-aware check, refusing whatever the category.
- interactions — D4's base-layer refusal departs from the pattern's warn rule unrecorded: fixed in D4 and the plan-writer note.
- interactions, walkthrough, bound-breaker — `--unset` of an old name refused, closing the removal route: fixed; `--unset` admits and removes it.
- existing-paths — `lym beam merge-parent` builds `run_settings` keys from the budget builders, which would become literal dotted keys: fixed in D8 with a nesting helper and a criterion.
- existing-paths, walkthrough, bound-breaker, interactions — D7's place in the walk, nested board writes and marker order unstated: fixed; D7 runs after the hub-wide write, idempotent with no marker.
- bound-breaker, walkthrough — files gaining old keys after the walk, and incomplete pairs: fixed; D5 and D7 state the bound and D4 names `lym config reconcile`.
- bound-breaker — a null old bare key: fixed; D5 drops and reports it.
- walkthrough — the recipe reference syntax check may admit one dot only: fixed in D8.
- interactions, existing-paths, bound-breaker — mixed-build risk names only the load refusal: fixed; the risk names local pruning and old-name writes.
- board-scope — `run_settings` and `lym config --set` may be meant to take one dotted token (class decision): recorded under Open risks for the parent; the design keeps the nested object.
- board-scope — the in-flight-peer risk names peers that add no loom key: fixed; the risk is generic.
- board-scope — "reinterpretation" read as the reconcile's move: rejected, informational; the reading stays in Auto-mode assumptions.
- leanness — Constraints' history of earlier moves: fixed; cut, kept here in Turn 2.
- leanness — Auto-mode assumptions repeat decisions: fixed; shortened, the `run_settings` line dropped.
- leanness — D2 rationale lists rejected groups: fixed; trimmed.
- leanness — D4 rationale argues against a rejected alternative: fixed; stated positively.
- leanness — D2 criterion pins an absence: fixed; now a Review bullet.
- leanness — `TopLevelValue` caller enumeration and a pointer sentence: fixed; rephrased and cut.

## Rejected alternatives

- Carry every layer by a `configreg.Reinterpretation`: it rewrites only the hub base file, flags worktree files and cannot reach a machine's untracked local files; a local file's old keys would be pruned first.
- Read old names as aliases at load: a lasting compatibility path of the kind 3d3b48127 removed, and it leaves two names for one key.
- Warn and ignore an old key: drops a tuned value silently.
- `run_settings` as dotted keys, `{"loom.code.max_bounces": 1}`: changes the board's shape and every reader for no gain; the nested object mirrors the file.
- A new `lym config --set loom.code.max_bounces=…` form: the module positional already names the file.
- Nesting drift's `judge_*` as `drift.judge.ask_parent`: contradicts the operator's example `drift.judge_ask_parent`.
- Nesting gateslot, landing, shed, shuttle and harden prefix groups: each file already names its area; no reader confusion is reported.
- `pr_review.ask_parent`: the entry names the area `pr`, and `pr.review_ask_parent` parallels `discussion.review_ask_parent`.
- A `PATTERN-config-layers` line also carrying the naming rule: over the rule-text cap, so the naming rule is its own entry.
- A once-per-hub board marker for D7: its commit and the board write cannot land together without nesting board writes, and an old-name entry written later would never be carried.
- Reusing `RetiredKeys` for old names: its prefix match covers the new area paths, and a `none` category admits the old key.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): do the unprefixed run-wide review keys nest under `review`? Yes; a bare `loom.judge` is the ambiguity the entry names.
- Q2 (auto-pick): `pr_review_*` area name? `pr`, keeping `review_` in the rest, per the entry's list of areas.
- Q3 (auto-pick): a bare model key that names an area? It becomes `<area>.model`.
- Q4 (auto-pick): which other templates nest? None besides `tapestry.yaml`, which shares the perch key builders with loom.
- Q5 (auto-pick): how are file layers carried? By a `configsync` move table on every reconcile, which also moves `profiles.yaml`'s blocks before it follows the template, and board `run_settings` by a step of every hub-wide walk.
- Q6 (auto-pick): what happens to an old name written later? Refused everywhere, naming the new full name.
- Q7 (auto-pick): old and new both set in one file? The new path wins and the dropped old key is reported.
- Q8 (auto-pick): how do Go messages render a key? Through one `configengine` helper from named constants.
- Q9 (auto-pick): does D7 use a marker? No; it runs idempotently on every hub-wide walk, after the hub-wide write.
- Q10 (open, for the parent): should `run_settings` or `lym config --set` take the dotted full name as one token? Recorded under Open risks.
