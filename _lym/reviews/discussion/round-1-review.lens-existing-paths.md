```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths — tracing every existing path that writes or reads the keys, files and board state the record changes.

**Paths traced, per decision**

- D4, `lym config reconcile` (no layer, `internal/configcli/configcli.go:565-596`): runs `ReconcileAll` on the worktree and local files only. It survives for worktree and local files and **breaks** as the way forward for the other sites.
- D4, `lym config reconcile --layer default` (`internal/configcli/hubwide.go:177-231`): runs `ReconcileHubWideAt` alone, with no reinterpretations and no board step. It **breaks** as a way forward for profile blocks and `run_settings`.
- D4, Seed-Child's `writeRunSettings` (`internal/battenshed/seamchild.go:146-157`): wraps a `PrepareRunSettings` refusal in its own way forward, `lym board upsert` or `lym config profiles`. It **breaks**: two contradictory ways forward.
- D4, `boardcli` upsert validation (`internal/boardcli/runsettings.go:144`): it survives, since old names are refused at the write.
- D4, the load's `warnUnknownKeys` (`internal/configengine/config.go:295-327`): it survives, since D4 replaces the warning with a refusal for moved keys only.
- D4, loom's `RetiredKeys` `pr_review` and `fan_review` and the prefix match `retiredCovering` (`internal/configengine/layers.go:206-220`, `internal/loomengine/config.go:733-749`): no new path falls under `pr_review.`. It survives.
- D5, `ReconcileAll`'s local-file prune (`internal/configsync/configsync.go:415-428`, `pruneUnknown=true`): it survives because the move comes before the prune.
- D5, `ReconcileHubWideAt` seeding an absent hub base from the prime's copy (`internal/configsync/configsync.go:470-476, 503-510`, `sparseSeed`): the prime's worktree file is moved only later, by `reconcileWorktree` (`internal/hubreconcile/walk.go:131-135`). It **breaks**, see F4.
- D5, the walk's commit of worktree files (`walk.go:329-339`, files that are `Applied` and in `LayerWorktree`): it survives if the move sets `Applied`.
- D5 and D8, the hub-wide reinterpretations run after `ReconcileHubWideAt` inside one write (`walk.go:221-244`): `FlagCodeAutoAccept` reads only the base file, which is already moved. It survives.
- D6, `FollowTemplate` merging `profiles.yaml` before the reinterpretation (`configsync.go:493-501`, `internal/yamlengine/follow.go:45-52`): it **breaks** for a shipped profile the operator edited, see F1.
- D7, `ensure`'s stamp write (`walk.go:77-84`) against D7's "retried by the next walk": it **breaks**, see F3.
- D7, `carryBeamTaskSettings` (`internal/hubreconcile/reinterpret.go:151-167`): beam keys only. It survives.
- D7, the detached `spawnBoardSync` started just before D7's board write (`walk.go:253`, `internal/boardengine/spawn.go:48`): D7's board write is pull-first (`internal/boardengine/registrysetter.go:167-190`). Whether they interleave is not verified, and I file no finding.
- D8, `CheckSettingKey` and `SettingLookup` read top-level `template[key]` and `snapshot[key]` (`internal/configreg/settinglookup.go:26, 64`); D8 states the nested lookup. It survives as decided.
- D8, the recipe reference `strings.Cut` at the first dot (`internal/shedrecipe/entries_settings.go:48`): it already passes a dotted path. It survives.
- D8, `lym beam merge-parent`'s settings literal (`internal/beamcli/mergeparent.go:82-84`): covered by D8's nesting helper. It survives.
- D8, `reviewbudget.Route` and `CheckLocal` (`internal/reviewbudget/reviewbudget.go:103-122, 181-210`): `CheckLocal` reads `set[b.key]` flat off a decoded document. D8's builder change returns a dotted key, and a flat lookup of `code.max_bounces` in a nested map misses. It **breaks** if left as is, see F5.

**Findings**

- **F1.** BLOCKING · design · `decision-record.md` D6, plus D5's collision rule. `ReconcileHubWideAt` runs `FollowTemplate` on `profiles.yaml` before D6's reinterpretation. Take a shipped profile whose `loom.code_max_bounces` the operator changed, say `tapestry` to 3:
  - its old key is the operator's and is kept;
  - the template's new `code` mapping is absent from both the file and the base, so the template's `code: {max_bounces: 1, …}` is added;
  - D6's move then collides with that path.

  Under D5's rule, which D7 says the shared helper uses, the new path wins, so the operator's value is dropped for the template's. D6's criterion tests only an operator-*added* profile.

  Evidence: `internal/yamlengine/follow.go:45-52`, `internal/configsync/configsync.go:493-501`, `internal/hubreconcile/walk.go:221-232`, `internal/runprofile/template.yaml:28-30`.
- **F2.** BLOCKING · design · `decision-record.md` D4, the way forward. D4 gives every refusal the same way forward, `lym config reconcile`, "which moves it". That command moves only worktree and local files. Each other site needs a different fix:
  - a hub base file needs `--layer default`;
  - a profile block is moved only by a once-per-hub reinterpretation that `--layer default` never runs;
  - `run_settings` is moved only by D7's walk step;
  - `--set` and a recipe reference need the new name.

  Seed-Child also appends its own way forward, `lym board upsert`, to a `run_settings` refusal, so the operator gets two conflicting ones. This violates `PATTERN-refusal-way-forward`.

  Evidence: `internal/configcli/configcli.go:582-595`, `internal/configcli/hubwide.go:194-224`, `internal/battenshed/seamchild.go:152-156`.
- **F3.** MEDIUM · design · `decision-record.md` D7, "retried by the next walk". `ensure` writes the build stamp whenever the walk was not skipped. D7's failure is only logged, so the stamp still goes current, and no walk runs again until the next build. No command runs D7 on demand. Until the next build, an entry holding old names stays stuck: Seed-Child refuses it, and D4's way forward does not reach it. The criterion "the next walk carries it" holds only across builds.

  Evidence: `internal/hubreconcile/walk.go:41-84`, and the `carryBeamTaskSettings` contrast at `internal/hubreconcile/reinterpret.go:151-167`, which uses a marker that stays unwritten on failure.
- **F4.** LOW · design · `decision-record.md` D5. Take an absent hub base `loom.yaml`: the hub-wide pass seeds it from the prime's worktree copy through `sparseSeed`, which prunes units the template lacks. The prime file is moved only afterwards, by `reconcileWorktree`. The prime's tuned old keys are therefore pruned from the seed, not moved into the hub base, and they live on only in the prime's tracked file. D5 does not say the move applies to the prime copy read as a seed.

  Evidence: `internal/configsync/configsync.go:470-476, 503-510, 278-296`, `internal/hubreconcile/walk.go:126-135`.
- **F5.** MEDIUM · design · `decision-record.md` D8, the `reviewbudget` bullet. `reviewbudget.CheckLocal` checks a decoded local document with flat `set[key]` lookups, and `PrepareRunSettings` hands modules a decoded nested object. With builders returning `code.max_bounces`, the flat lookup misses the nested `code: {max_bounces: …}`, and every local budget bound check silently passes. D8 nests the writers (merge-parent) and the lookup (recipe references), but never states that `CheckLocal` and other flat-map readers of local documents resolve the dotted path.

  Evidence: `internal/reviewbudget/reviewbudget.go:181-210`. I have not read `loomengine.ValidateLocalValues`, the `CheckValues` caller at `internal/configreg/configreg.go:164`.

**Tool calls**
1. I grepped for production `run_settings` builders and read `SpawnSync`'s command, to find other writers of `run_settings` and the board sync process. The context held neither.
2. I read `internal/battenshed/seamchild.go:135-200`, Seed-Child's run-settings write and its way forward. The context showed only the call site.
3. I read `internal/boardengine/registrysetter.go:160-200` and the board `Sync` entry points, to learn how a board write pulls and pushes. The context held only their names.
