```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the existing code paths that write or read each state the record's decisions change.

**Paths traced, per decision**
- D2, `loomrecipe.PastDiscussionApproval` over tapestry with Preflight moved after the Motif block: survives. Preflight now indexes past the approval segment, which matches a post-Discussion row. The in-flight run parked at Preflight that never ran Motif now reads as past approval. Its board copy fails on the missing committed record, and `copyRecordToBoard` only warns about that, so it is harmless (decision.go:697-704, 712-735).
- D2/D4, the `undiscussed` block gate seam seeding a discussed tapestry entry, then Record-Seed: survives. The gate writes first (entries_block.go:48-64) and Record-Seed finds both files and writes nothing.
- D4/D6, `boardCopyDue` and `decision store` on a jacquard run, which has no approval row: survives. The `!approvalRow` branch copies when `boardRecordStored`, and a filed or discussed jacquard entry has a stored record (decision.go:697-704, 1351-1358).
- D4, `rowWritten` reading `RecordEngine == "RecordWrite"`: breaks unless changed. The record covers it with "decision verbs treat RecordSeed as they treat RecordWrite" (decision.go:1724-1731).
- D4, `recordWritePending`, a literal Preflight or Record-Write: covered by D4's EngineRow clause (decision.go:604-606).
- D5, `lym loom step`'s bootstrap: breaks the claim. `step` calls `seedAndCommitBootstrap` (internal/loomcli/arm.go:636), which seeds the status file on a first step just as `start` does (start.go:572). D5 names only `lym loom start` and does not say whether the check sits in the shared bootstrap.
- D5, the dobby task view's start refusal: survives. `startClearsMotifRun` is shared with internal/battencli/dobbyreads.go:490-498, so the widened clear also stops the view showing `doneSlugRefusal` for a jacquard entry over a done motif run. A jacquard entry with no stored record then reaches batten's guard, as a raddle entry does today.
- D5, `ClearMotifRun` versus the leftover task branch: survives as far as read. Fabric's `Remove` deletes the local task branch when its work is on another ref, and deletes the origin branch through the landed-work gate (internal/fabricengine/remove.go:76-90). A motif pair commits no code, so a done motif run is unlikely to leave a branch that Worktree-Create's `ErrBranchExists` refuses (internal/battencli/wire.go:381-393). This is not verified end to end for the origin half.
- D5, the batten-site way forward over a done motif run (`lym batten cleanup --apply`, then upsert tapestry): survives on the same reading. `Cleanup` touches no branch (cleanup.go:111-114), so the start depends on the teardown having removed the branch.
- D7, reconcile follow, then reinterpretation: survives. The hub-wide walk applies reinterpretations after `ReconcileHubWideAt` in the same write (internal/hubreconcile/walk.go:250-273), so a `defaults.jacquard` gained by the follow is there for the one-time drop. The follow adds `defaults.jacquard` on a hub that removed `solid`, and keeps `solid` removed (follow.go:156-188).
- D7, a shipped profile whose tapestry block the operator tuned: breaks silently. The follow gains `&lt;profile&gt;.jacquard` with the template's values, because both the file and the base lack the key (follow.go:173-181). The reinterpretation copies from the tapestry block only for an operator-added profile. A raddle part that ran a tuned tapestry block before D6 runs untuned jacquard values after it, and the record says nothing about this.
- D9, `runUsageRecord` and `writeRunUsage`: survives. The recipe is read from the seed and `profiles.Default("jacquard")` resolves once the defaults are gained (sharedbootstrap.go:205-247).
- D3, `skipTarget` over a last block with gate `always`: survives. The gate never skips (producer.go:132-137, entries_block.go:44-46).

**Findings**
1. MEDIUM, design, `_lym/discussion/decision-record.md:145`: D7 leaves an operator's tuning of a shipped profile's tapestry block out of the gained jacquard block. The reconcile adds `&lt;profile&gt;.jacquard` at the template's values, and the reinterpretation derives a block from tapestry only for operator-added profiles. Raddle parts that ran tuned tapestry settings silently run shipped jacquard settings after D6. Evidence: internal/yamlengine/follow.go:173-181 (absent in the file and base takes the template's value); decision-record.md:148-149.
2. LOW, consistency, `_lym/discussion/decision-record.md:110`: D5 judges the first-start check in `lym loom start` only. `lym loom step` reaches the same first seed through `seedAndCommitBootstrap` (internal/loomcli/arm.go:636; sharedbootstrap.go:128), and the record does not say whether step is checked. Record-Seed's Stuck still stops such a run. Evidence: internal/loomcli/arm.go:636, internal/loomcli/start.go:572.

**Tool calls**
- Grep for `seedAndCommitBootstrap` callers, the dobby view and the loomrecipe approval helpers: the manifest lacked the step path and the dobby's use of `startClearsMotifRun`.
- Grep of teardown and remove for task-branch handling: needed to judge whether a done motif run leaves a task branch.
- Read of internal/fabricengine/remove.go:75-110, 575-600: the task-branch deletion contract.
- Grep in decision.go for `pastApproval` and `storeRecord`: the board-copy path a jacquard run takes.
- Read of decision.go:685-735, 1340-1375: `boardCopyDue` and the store verb's approval branches.
