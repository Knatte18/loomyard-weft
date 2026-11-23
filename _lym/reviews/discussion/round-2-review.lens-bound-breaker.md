```yaml
lens: bound-breaker
outcome: reported
```

**bound-breaker: claims attacked**

- D1, the key counts ("all 18", "all 13", "all 8", "all 6"): held. The template has 18 `discussion_`, 13 `code_`, 8 `split_` and 6 `review_` top-level keys.
- D1, tapestry `part_writer` → `parts.writer` against the other `parts_*` keys: held. No `parts_writer` key exists, so nothing collides, and `parts_chair` becomes `parts.chair`.
- D4, an old bare area key matches only a non-mapping value: held. A run_settings or file value `discussion: {model: …}` is judged as new, and `discussion: sonnet` as old.
- D4, the old-name check stays apart from loom's `RetiredKeys` prefix match (`fan_review`, `pr_review`, the shed-only and tapestry plan keys): held. `pr_review.` is not a prefix of `pr.review_*`, and no retired key is a prefix of a new area.
- D4, `--unset` admits an old name: held. `configengine.Unset` already removes any key, including one the template lacks (`internal/configengine/set.go:111`).
- D4, Seed-Child prints one way forward: **broken** (finding 2).
- D4, the hub base way forward `lym config reconcile --layer default --apply` reaches the agent that gets the refusal: **broken for a task session** (finding 3).
- D5, "the reconcile … visits every file layer … before any command runs" and the bound "once per build per machine": **broken** (finding 1).
- D5, a file holding both an old key and its new path keeps the new path, and a tuned value is never lost: **broken under mixed builds** (finding 4).
- D5, the move is idempotent with no marker: held. A moved area is a mapping, so it is never read as an old key; a null old key is dropped.
- D5, an absent hub base is seeded from the prime copy after the move: held as stated. `sparseSeed` prunes, and the move is placed before it.
- D6, the move runs before the follow, and the collision with a template-added new path: held. With file and base moved first, a moved operator value differs from the moved base, so `followUnit` keeps it (`internal/yamlengine/follow.go:167-171`).
  - The mixed-build path also holds for profiles. An older build's follow re-adds the flat key at the template default, the newer walk sees both, and the new path holds the operator's value.
- D7, a failed write keeps the stamp stale and the next start verb retries: held. `ensure` writes the stamp only when the walk is not skipped (`internal/hubreconcile/walk.go:81-85`).
- D7, the rewrite reaches run-held entries: held. `scopeChanged` never compares `RunSettings` (`internal/boardengine/runlock.go:211-213`).
- D8, `FlagCodeAutoAccept` runs after the move: held. It ignores the worktrees argument and reads only the hub base, which `ReconcileHubWideAt` moves first (`internal/loomengine/reinterpret.go:27`, `internal/hubreconcile/walk.go:228-243`).
- D5 bound, the machine layer holds no loom or tapestry key: held in the loom spec span I read (`internal/loomengine/config.go:690-732`); no machine category appears there.

**Findings**

1. MEDIUM, design, `_lym/discussion/decision-record.md:118` (D5 bound) and line 120 (D5 rationale)
   - Summary: the walk runs only from a start verb, so every other verb an in-flight run calls on the new build loads its files before the walk moves them. D4's refusal then halts the run.
   - The rationale's premise is false: the walk does not visit every file layer "before any command runs". `hubreconcile.Ensure` is called only by start verbs: loom `start` and `resume`, batten arm, beam start, adopt, preflight and clone.
   - Counterexample: lym is rebuilt while a run is past Seed-Child. Its next verb, such as a gate or validate verb, is not `start` or `resume`, so no walk runs. The verb strictly loads a hub base `loom.yaml` that still sets `code_max_bounces: 2`, or a local file Seed-Child wrote from a shipped profile block. D4 refuses the load, where the hub base today only warns. Every in-flight run on the hub halts until someone runs a start verb or the reconcile.
   - D5's bound names only files that gain an old key after the walk, not files the walk has not reached yet on the new build.
   - Evidence: `internal/loomcli/arm.go:219-225` reconciles only for start and resume; `internal/configengine/config.go:302-329` shows the base layer warns today; Ensure callers from the grep (`beamcli/cli.go:539`, `preflightshed/preflight.go:207`, `loomcli/arm.go:223`, `battencli/arm.go:302`, `battencli/wire.go:351`, `adoptcli/check.go:441`, `fabriccli/clone.go:153`).
2. MEDIUM, design, `_lym/discussion/decision-record.md:91-98` (D4)
   - Summary: the local-file refusal and the run_settings refusal share one validator, so a run_settings refusal can carry the local file's way forward, and Seed-Child then prints two.
   - D4 says the local-file refusal names `lym config reconcile --apply`, and the run_settings refusal through `PrepareRunSettings` names only the old and new names.
   - Both paths run `configengine.ValidateLocal`: the local load does, and `PrepareRunSettings` reaches it through `checkLocalDocument`.
   - D4 does not say that the run_settings check runs before `checkLocalDocument`, as `refuseRetiredKeys` does today.
   - Counterexample: the check is put in `ValidateLocal`, where the local-file wording lives. `{"loom":{"code_max_bounces":2}}` at Seed-Child is then refused with "run `lym config reconcile --apply`", which cannot fix a board entry, and Seed-Child's wrapper adds `lym board upsert`. That is the two-way-forward defect F1 closed.
   - `lym config loom --layer local --set code_max_bounces=2` has the same gap: `setSparse` refuses through `Unknown`, and then `validateOverride` runs `ValidateLocal`, before any `--set`-specific refusal.
   - Evidence: `internal/configreg/runsettings.go:63-75` and `144-152`; `internal/configengine/set.go:206-221`; `internal/battenshed/seamchild.go:154-155`.
3. MEDIUM, design, `_lym/discussion/decision-record.md:90` (D4, hub base site)
   - Summary: a task session that gets the hub base refusal is refused the command its way forward names.
   - The refusal is raised by a strict loom load in any session, most often a task's driver or agent.
   - `lym config reconcile --layer default --apply` is refused from a task session or a non-prime worktree; that refusal sends the fix to the operator or the hub orch.
   - The receiving agent is stranded unless the way forward says who runs it, as `hubWriteRefusal`'s own text does.
   - Evidence: `internal/configcli/hubwide.go:51-63` and `202-208`; `internal/configcli/configcli.go:584-585`.
4. MEDIUM, design, `_lym/discussion/decision-record.md:114` (D5 collision rule) against Goal line 8
   - Summary: under mixed builds, the newest value an operator sets is dropped for a stale one.
   - Counterexample:
     1. The new build's walk moves the hub base `loom.yaml`, which now holds `code.max_bounces: 2`.
     2. An operator on a machine still on the older build runs `lym config loom --layer default --set code_max_bounces=3`. The older template knows that key, and `setSparse` keeps the unknown `code` map as preserved.
     3. The new build refuses the file until the walk or `lym config reconcile --layer default --apply` runs.
     4. The reconcile keeps the new path's 2 and drops the operator's newer 3.
   - The drop is reported, but the Goal's "existing values carry over" fails, and the mixed-build Open risk names old-name writes without this loss.
   - Evidence: `internal/configengine/set.go:200-229`; `internal/configsync/configsync.go:497-508`.

**Tool calls**

- Counted the prefixed keys in `internal/loomengine/template.yaml` and the `parts`/`plan` keys in `tapestry.yaml`: to test D1's counts and the `parts.writer` collision; the templates were not in my context.
- Grepped `FlagCodeAutoAccept` for worktree reads: to test D8's ordering claim; the function was not in my context.
- Read `internal/loomcli/arm.go:195-239`: to confirm which loom verbs run `Ensure`; only the call line was in my context.
