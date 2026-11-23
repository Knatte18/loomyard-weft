```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I paired the record's decisions that act on the same state and checked the acceptance criteria and constraints against them.

**Pairs checked**
- D1 × D4 (a bare old key like `discussion`, `raddle`, `friction`, `plan` or `parts` is told from its new map by shape): held. A null or scalar value is the old key and a mapping is the new area.
- D4 × `RetiredKeys` (loom's `pr_review`, `fan_review` and the tapestry plan keys): held. The prefix match needs `pr_review.`, so it covers neither `pr.review_*` nor `pr_review_*`.
- D4 × D5, hub base file refused at load: broken. D4's way forward `lym config reconcile` does not reach the hub base file (finding 2).
- D4 × D5, worktree and local files: held. `lym config reconcile` with no layer runs `ReconcileAll` over them.
- D4 × D6, profile block refused: broken. The way forward cannot move a profile key once D6's marker exists (finding 2).
- D4 × D7, `run_settings` refused at Seed-Child: broken. No reconcile verb runs D7's board step (finding 2).
- D4 `--unset` × D5: held.
- D5 × D6 (D6 runs after `configsync`'s FollowTemplate merge of `profiles.yaml`): broken. When the operator edited a value in a shipped profile, the old key and the new path collide (finding 1).
- D6 × D7 (shared `configreg` helper): broken. Neither decision states what the helper does on a collision (finding 1).
- D5 × D8 (`FlagCodeAutoAccept` reads `code.auto_accept` after the move): held. Reinterpretations run after `ReconcileHubWideAt` inside the hub-wide write, and `FlagCodeAutoAccept` reads only the hub base file.
- D7 × D8 (`lym beam merge-parent` builds nested `run_settings`): held. D7 is idempotent on nested objects.
- D8 × D9 (reviewbudget builders × dotted categories, longest match): held.
- D9 × the D4 criterion (`discussion: {reviewer_fan: ""}` is a Guard key): held.
- D3 × the D4 criterion (`plan_max_bounces` names `tapestry.plan.max_bounces`): held.
- D7's retry claim × D5's once-per-build bound: broken. D7 says a failed write is "retried by the next walk", but the next walk comes only with the next build, while D4 refuses the entry meanwhile (folded into finding 2).
- D6 × its own criterion ("equals the new template"): weak (finding 3).
- Constraint "`TopLevelValue` reads top-level keys only" × D2: held. Its only caller is `dobbyengine` (`internal/dobbyengine/tasksetting.go:59`).
- Constraint "walk order" × D5, D6, D8: held. In `internal/hubreconcile/walk.go`, `ReconcileHubWideAt` runs before `applyReinterpretations` in the same `BoltWrite`.

**Findings**

1. **BLOCKING · design · D6 with D7.** An operator-edited shipped profile loses the operator's value.
   - D6 runs after `configsync`'s merge. `yamlengine.FollowTemplate` descends into the recipe's `loom` map (`internal/yamlengine/follow.go:45-49`).
   - Take a shipped profile where the operator changed `loom.code_max_bounces` from 1 to 3. The old key is kept as operator-owned, because the file differs from the base.
   - The template's new `loom.code` map is added, because the file and the base both lack it and so match.
   - D6 then moves the old key onto a new path that already holds the template's value. D6 is silent on this collision.
   - D7's shared helper is also silent, and D5's rule ("keeps the new path's value, drops the old key") applied here drops the operator's 3. That breaks the Goal's "Every hub's existing values … carry over".
   - D6's criterion tests only a profile the operator added, which has no template counterpart.
   - Evidence: `internal/runprofile/template.yaml:28-69` (shipped profiles hold `loom: {code_max_bounces…}`); `internal/configsync/configsync.go`, the `ReconcileHubWideAt` seed-only branch (FollowTemplate runs before any reinterpretation).
   - Fix: state the helper's collision rule for D6. Either the old key's value wins when it differs from the template base, or the move runs before the follow, against a moved template base. Add a criterion for an edited shipped profile.

2. **BLOCKING · design · D4 with D5, D6 and D7.** D4's one way forward, "`lym config reconcile`, which moves it", is unreachable at most of the sites it refuses. This breaks `PATTERN-refusal-way-forward`.
   - Hub base file: `runReconcile` with no layer runs only `ReconcileAll` over the worktree and local files. Moving the base needs `--layer default` (`internal/configcli/configcli.go:565-593`).
   - Profile block: `reconcileDefaultLayer` runs only `ReconcileHubWideAt` and never a reinterpretation. Once D6's marker exists, no reconcile moves a profile key (`internal/configcli/hubwide.go:177-231`; `internal/hubreconcile/reinterpret.go:51-55`, where the marker skips the change).
   - Task `run_settings`: no reconcile verb runs D7's board step. D7 runs only on a hub-wide walk, which the build stamp gates to once per build (`internal/hubreconcile/walk.go:39-46`).
     So D7's "a failed read or write is logged and retried by the next walk" leaves the entry refused at Seed-Child until the next build, with a way forward that does nothing.
   - `lym config &lt;module&gt; --set` and a recipe setting reference: a reconcile cannot fix either. The way forward there is the new name, or editing the recipe.
   - Fix: name the way forward per site. Examples: `lym config reconcile --layer default` for the hub base, `lym board upsert` with the new name for `run_settings`, an edit of `profiles.yaml` for a profile block, and the new name for `--set` and a recipe reference.

3. **LOW · consistency · D6 acceptance criterion.** "an untouched seeded `profiles.yaml` equals the new template" cannot hold byte for byte.
   - FollowTemplate re-encodes a changed file, drops the blank lines between entries, and copies a template unit in with none of its comments (`internal/yamlengine/follow.go:53-57`).
   - Fix: state the criterion as equal as parsed YAML.

Tool calls:
1. `grep` for `TopLevelValue(` callers, to check the Constraints claim that its readers stay flat. My context had only its definition, not its callers.
