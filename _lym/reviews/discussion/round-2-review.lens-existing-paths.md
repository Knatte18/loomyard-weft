```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens for round 2 of the discussion review. I traced every existing path that writes, re-applies or reads the state D4 to D9 change.

**Paths traced, per decision**

- D4, local-file refusal: Seed-Child writes the local file from `run_settings` (`internal/battenshed/seamchild.go:146-177`). That goes through `PrepareRunSettings` first, which refuses old names, so no old key gets in. **Survives.**
- D4, local and worktree way forward: `lym config reconcile --apply` runs `ReconcileAll` with `PruneUnknown: true` (`internal/configcli/configcli.go:590`). With D5 inside configsync, it moves the old key. **Survives.** Side effect: the same run also prunes the tracked worktree file's other unknown keys, which `PATTERN-config-layers` already allows for the operator's reconcile.
- D4, hub-base refusal and the `--layer default` way forward: `reconcileDefaultLayer` runs `ReconcileHubWideAt` with `SeedDefaults` and `PruneUnknown` (`internal/configcli/hubwide.go:192-223`). With D5 inside configsync, it moves the key. **Survives.**
  - `hubWriteRefusal` refuses that command from a task session or a non-prime worktree (`internal/configcli/hubwide.go:51-62`).
  - So a loom run in a task worktree that hits the load refusal cannot follow the way forward itself; the operator or the hub orch has to.
  - This matches the existing `warnUnknownKeys` text (`internal/configengine/config.go:310`). It is not on the closed list of denied forms.
- D4, hub-base strict validation: `setHubWide` and `copyStagedToHub` load the board file strictly (`internal/configcli/hubwide.go:325-327`, `:466`). While the hub base still holds an old key, they refuse every `--layer default` set or edit of loom. The way forward is the same reconcile. **Survives**, and D5's bound covers it.
- D4, `run_settings` through `lym board upsert`: `validateRunSettings` uses `PrepareRunSettings` (`internal/boardcli/runsettings.go:144`). **Survives.**
- D4, profile block: `checkProfiles` uses `PrepareRunSettings` (`internal/configreg/runsettings.go:156-169`). Editor refusals go through `hubFileLoadRefusal`, which names a way forward (`internal/configcli/hubwide.go:477-479`). At Seed-Child, `LayerRunSettings` is wrapped in `ErrInvalidRunSettings` (`internal/battencli/runsettings.go:49-54`), and Seed-Child adds its own way forward (`seamchild.go:155`). **Survives.**
- D4, `--set`: `Set` and `setSparse` refuse a key the template lacks as unknown (`internal/configengine/set.go:96-98`, `:210-212`). D4's own check has to come first so the new path gets named. **Survives** as stated.
- D4, `--unset`: `Unset` already removes any key, unknown ones included (`internal/configengine/set.go:111-133`). **Survives.**
- D4, recipe reference: `settingRef` cuts at the first dot (`internal/shedrecipe/entries_settings.go:48`). `CheckSettingKey` reads top-level template keys only (`internal/configreg/settinglookup.go:23-28`). D8 changes both. **Survives** as stated.
- D5, every configsync caller: `ReconcileAll` and `ReconcileHubWideAt` are called from the walk (`walk.go:228`, `:332`), the reconcile verbs (`configcli.go:590`, `hubwide.go:195-223`), clone (`internal/fabriccli/clone.go:71`, `:120`) and `fabric.go:677`. All of them go through configsync, so the move covers every one. **Survives.**
- D5, commit of a moved worktree file: `reconcileWorktree` commits only results that are `Applied` at the worktree layer (`walk.go:346-363`). `reconcileSparseFile` sets `Applied` only when `Removed` is non-empty (`configsync.go:334-344`). D5 says the walk commits a moved file, so the move must set `Applied`; that is plan detail. **Survives.**
- D5, hub-base seed from the prime copy: `primeConfig` then `sparseSeed` (`configsync.go:449-493`). D5 now moves before the prune. **Survives.**
- D5 against the stamp: `ensure` returns early while the stamp is current (`walk.go:43-46`). An old key that arrives later waits, and D5's bound states this. **Survives.**
- D6, readers of `profiles.yaml`: `runprofile.Load` at Seed-Child (`battencli/runsettings.go:26`) and `checkProfiles` both read the moved file. The template base is rewritten whenever it is stale (`configsync.go:543-552`). The dry run of `reconcileDefaultLayer` moves only in memory. **Survives.**
- D7 against `carryBeamTaskSettings`: that carry reads only the beam module of `run_settings` (`internal/boardengine/store.go:403-409`). D2 leaves beam alone. If D7 marks the walk skipped, this carry is also skipped in that walk (`walk.go:162-164`), but it keeps its own marker and retries. **Survives.**
- D7 against entries held by a run: `scopeChanged` never compares `RunSettings` (`internal/boardengine/runlock.go:211-213`), so a running entry accepts the rewrite. **Survives.**
- D7 retry: a skipped walk leaves the stamp unwritten (`walk.go:81-84`). The log line there names only a skipped hub-wide write or a mid-merge worktree, so it would misdescribe a D7 failure. That affects the log only. **Survives.**
- D8, existing reinterpretation `FlagCodeAutoAccept`: it reads only the hub base bytes it is handed (`internal/loomengine/reinterpret.go:27-50`). `applyReinterpretations` runs after `ReconcileHubWideAt` in the same write (`walk.go:228-243`), so the order D8 relies on holds. **Survives.**
- D8, `autonomySettingLookup`: it matches loom keys by name through `loomengine.AutonomyForcedSetting(key)` (`internal/loomcli/wiring.go:1413-1426`). It becomes a reader of the dotted path D8 introduces, and D8's rule that every reader uses the nested path covers it. **Survives.**
- D9, `ValidateLocal` and `retiredCovering`: the retired `pr_review` prefix does not cover `pr.review_*` or `pr_review_*` (`internal/configengine/layers.go:206-218`). **Survives.**

**Findings**

None. No existing path undoes or misreads a decision while the record stays silent about it.

The nearest item is the hub-base way forward under D4: an agent in a task session gets the refusal but cannot run the command it names, because `hubWriteRefusal` refuses it there (`internal/configcli/hubwide.go:51-62`). Severity LOW, class design, location `_lym/discussion/decision-record.md:90`. It matches the existing `warnUnknownKeys` wording, it is not BLOCKING, and this incremental round's rule therefore keeps it from being raised as a finding.

**Tool calls**

1. Read `internal/loomengine/reinterpret.go` (`FlagCodeAutoAccept`): a reinterpretation reader of a moved key, outside the manifest.
2. Read `_lym/reviews/discussion/round-2-changes.md` and the git log: to know which files changed this round, which the incremental rule needs.
3. Read `autonomySettingLookup` in `internal/loomcli/wiring.go` and grepped `raddlefile.go`: an existing reader of loom keys by name, and a writer of `run_settings`, both outside the manifest.
