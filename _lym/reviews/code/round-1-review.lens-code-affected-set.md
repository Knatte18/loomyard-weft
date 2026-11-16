```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set. I checked each change the diff makes against what calls, reads, asserts or documents it.

**What each change touches, and whether the diff keeps it right**

- **`landDeps.ParentConflicts`, the new field** (`internal/beamcli/land.go:95`).
  - The struct is built in only two places: `hubLandDeps` (`internal/beamcli/deps.go:180`) and the test fake `landWorld.deps` (`internal/beamcli/land_test.go:133`, with the new field at line 207). Both fill the field, so neither leaves it nil. Covered.
  - Its field doc (`land.go:58`) and the package doc (`doc.go:210-211`) are both updated. Covered.
- **`beamParentTips`, pulled out of `ParentUnmerged`** (`deps.go:147-179`).
  - Callers: `ParentUnmerged` and `ParentConflicts`.
  - The behaviour `lym beam ready` relies on through `judgeParentMerged` (`land.go:222`) keeps the same read order and the same fetch condition.
  - `TestHubLandDeps_ReadyReads` (`deps_integration_test.go:69-89`) still asserts it. Covered.
- **`fabricengine.Fabric.ConflictsWith`, the new verb** (`parenttips.go:219-236`).
  - Its only caller is `ParentConflicts`.
  - The verb is general (one ref merged into HEAD), so it fits PATTERN-fabric-boundary's "Fabric isolation" rule.
  - The git it runs goes through `gitrepo.MergeTree`, which is already on the pinned list (PATTERN-gitrepo-client-boundary).
  - `internal/fabricengine/doc.go` documents none of this file's sibling verbs (`HoldsWork`, `HeadHoldsCommitsNotIn`), so leaving it out of that doc is consistent. Covered.
- **`prOpen` / `openRecorded` gain the `judgeOpenable` refusals** (`pr.go:81`, `pr.go:140`). Where this behaviour is described:
  - The `prOpen` doc comment (`pr.go:51`): updated.
  - The `beamcli` package doc (`doc.go:161-166`): updated.
  - The orch role contract (`contracts/orchrole/role-beams.md:39-40`): updated.
  - The `lym beam pr open` cobra help (`internal/beamcli/cli.go:81-89`): **not updated**. See finding 1.
  - `internal/landingshed/doc.go:647`, `internal/pawlengine/retiredmove.go:176` and `docs/overview.md:817` name the verb without listing its refusals, so they are unaffected.
  - `lym beam pr update`'s help (`cli.go:111-114`) says it refuses what `pr open` refuses "of the entry, the owners, the run and the ready record". That scope stays correct, because update does not run `judgeOpenable`.
- **The two new refusal messages** (`pr.go:247`, `pr.go:254`).
  - The `TestPROpen` rows at `pr_test.go:128-151` assert them, and each names its way forward.
  - Both commands they name (`lym batten status`, `lym beam merge-parent`) are lym verbs no settings deny, so PATTERN-refusal-way-forward holds. Covered.
- **`openMergeTask`, the existing helper now called from `judgeOpenable`.**
  - Its contract is unchanged, and `mergeparent_test.go:61` already pins its done-task case. Covered.
- **The commit messages over the range.**
  - The claim "before it pushes … or reopens a closed one" matches `pr.go:81` and `pr.go:140`.
  - No commit carries a `Pattern-Departure:` trailer, and the directive declares no PATTERN changes, so nothing is owed there. Covered.

**Findings**

1. **MEDIUM · consistency** · `internal/beamcli/cli.go:81-89`
   The `lym beam pr open` help lists every refusal ("refuses, each naming its way forward:"), but it does not list the two new ones: a merge task that is not done, and a beam that conflicts with its parent branch.
   An operator or orch reading `--help` gets an incomplete list, and the package doc (`doc.go:161-166`) and `role-beams.md:39-40`, which the diff did update, now disagree with it.
   Evidence: `cli.go:81-89`, read against `pr.go:241-257`.

**Tool calls**

1. `grep` over the docs and non-test Go files for `HoldsWork`/`HeadHoldsCommitsNotIn` and `beam pr open`: to find the doc, stencil and contract mentions of the changed verb and of the sibling fabric verbs, which my context did not list.
2. `grep` of the `beam pr open` mentions in `landingshed/doc.go`, `retiredmove.go` and `overview.md`, of fabricengine's `doc.go` coverage of the parenttips verbs, and of every `landDeps{` construction: to check whether those places enumerate refusals or verbs, and that every construction fills the new field.
