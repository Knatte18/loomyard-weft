```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: attack each bound, ordering promise and claim about existing code that the record states.

**Claims attacked**

1. D5: "the walk runs once per build per machine". **Held.** The stamp is under the board's untracked scratch dir (`internal/hubreconcile/hubreconcile.go:27-28`), and a pair call reconciles more often, which is harmless.
2. D5: the move only touches keys the table names and never the machine layer, "which holds no loom or tapestry key". **Held.** Loom's `Categories` hold no `CategoryMachine` key (`internal/loomengine/config.go:679-731`), and reviewbudget adds only none or guard.
3. D5: "Moving before the prune keeps a local file's old key from being pruned". **Broken for the hub base seed** (finding 4).
4. D4: the refusal names `lym config reconcile`, "which moves it", as the way forward at every site. **Broken** (finding 1).
5. D7: "a failed read or write is logged and retried by the next walk". **Broken in effect** (finding 2).
6. D7 bound: "Seed-Child refuses it meanwhile under D4, naming the new name". **Held** for the refusal itself. Its way forward is broken (finding 1).
7. D6: "It runs in the walk after configsync's merge, so a profile the operator never changed has already taken the new template". **Broken** for a shipped profile the operator edited (finding 3).
8. D7: "Running outside the hub-wide write keeps the two board writes from nesting". **Held** as far as I read. The hub-wide write finishes its `PullThenCommitWritten` before `spawnBoardSync` (`walk.go`, `reconcileHubWide`). A board write through `commitBoardWriteAndPush` is its own pull-first Bolt commit (`internal/boardengine/registrysetter.go:167-190`). Whether it races the detached board sync that the hub-wide pass just spawned is not stated or pinned. I did not break it.
9. D8: "FlagCodeAutoAccept reads code.auto_accept, since every reinterpretation runs after configsync's move". **Held.** `ReconcileHubWideAt` runs before `applyReinterpretations` inside the same write (`walk.go`, `reconcileHubWide`), and `FlagCodeAutoAccept` reads only the hub base bytes (`internal/loomengine/reinterpret.go:27`).
10. Constraint: the walk runs configsync, then reinterpretations, inside one hub-wide write, and each worktree's reconcile commits what it wrote. **Held** (`walk.go`, `reconcileHubWide` and `reconcileWorktree`).
11. D4: in a hub base or worktree file, "a key the template lacks otherwise only warns". **Held** (`internal/configengine/config.go:295-327`).
12. D4 and `RetiredKeys`: loom's retired `pr_review` matches only `pr_review` and paths below `pr_review.` (`internal/configengine/layers.go:206-218`). It covers neither `pr_review_ask_parent` nor `pr.review_*`. **Held.**
13. D8: "its syntax check admits a dotted path". **Held.** The current code already cuts at the first dot (`internal/shedrecipe/entries_settings.go:48`). `CheckSettingKey` and `SettingLookup` read top-level keys only (`internal/configreg/settinglookup.go`), and D8 changes both.

**Findings**

1. **BLOCKING, design, D4 (way forward).** D4's one way forward, `lym config reconcile`, "which moves it", does not move the old key at most of the sites it refuses. This breaks `PATTERN-refusal-way-forward`, which requires the refusal to name a way forward that a test reaches.
   - **Hub base file:** `lym config reconcile` with no `--layer` runs only `configsync.ReconcileAll` on the worktree and its local files (`internal/configcli/configcli.go:584-590`). The hub base needs `--layer default`.
   - **Profile block:** `--layer default` runs only `configsync.ReconcileHubWideAt` (`internal/configcli/hubwide.go`, `reconcileDefaultLayer`). D6 carries `profiles.yaml` by a reinterpretation, whose board marker is already written after the first walk (`internal/hubreconcile/reinterpret.go`, `applyReinterpretations`). So no reconcile ever moves an old key that lands in a profile later.
   - **`run_settings`:** no reconcile verb runs D7's board step, so Seed-Child's refusal points at a command that leaves the entry unchanged.
   - **`--set` and recipe references:** reconcile does not apply, because the fix is to use the new name.
2. **MEDIUM, design, D7 (retry bound).** "Retried by the next walk" means the next build on that machine. A failed D7 board write is only logged, and the build stamp is still written unless the walk was skipped (`walk.go`, `ensure` → `writeStamp`). Counterexample: the board push is refused once, the stamp is written, and every not-yet-started old-name entry stays unmoved until another lym build. Seed-Child refuses it meanwhile, with the unusable way forward from finding 1. The D7 criterion "the next walk carries it" hides that gap.
3. **BLOCKING, design, D6 (shipped profile the operator edited).** `profiles.yaml` is seed-only and is reconciled with `yamlengine.FollowTemplate` at unit level, descending into mappings (`internal/yamlengine/follow.go:41-60`). Counterexample: the operator edited the shipped `solid.tapestry.loom.code_max_bounces` from 1 to 3. The follow keeps `code_max_bounces: 3` as the operator's unit and adds `code: {max_bounces: 1, hard_max_bounces: 2}` from the template, because file and base both lack `code`. D6's reinterpretation then moves `code_max_bounces` onto an existing `code.max_bounces`. Under D5's collision rule, which keeps the new path and drops the old (D7 says D6 shares the helper), the operator's 3 is dropped for the template's 1. D6's criterion tests only an operator-added profile, which never collides.
4. **LOW, design, D5 (prime seed).** When the hub base `loom.yaml` is absent, `ReconcileHubWideAt` seeds it from the prime's worktree copy through `sparseSeed`, which prunes units the template lacks (`internal/configsync/configsync.go`, `ReconcileHubWideAt` and `sparseSeed`). That happens before the prime's own worktree reconcile moves its keys (`walk.go`, `walkHub` order). Counterexample: a new hub whose prime commits `code_max_bounces: 2`. The seeded hub base loses the value, and it survives only in the prime's worktree layer. D5 does not say whether the move applies to seed bytes read from the prime.

**Tool calls**

1. Read `internal/boardengine/registrysetter.go:150-200` and the hubreconcile stamp and marker paths. My context lacked the board write path's pull, commit and push behaviour (for the D7 nesting claim) and whether the build stamp is per machine (for the "once per build per machine" bound).
