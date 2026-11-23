```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case. Two findings break a decision's purpose, and three smaller ones follow.

**Findings**

1. **BLOCKING — D4 with D1: the derived retired keys cover the new area maps.**
   - D4 derives the retired-key refusal from the move table, so the old bare keys `discussion`, `raddle`, `friction`, `plan` and `parts` become retired keys.
   - `retiredCovering` in `internal/configengine/layers.go` matches any path equal to a retired key or starting with it plus a dot. So retired `discussion` covers every new path such as `discussion.reviewer_fan` and `discussion.lenses`.
   - Case: a local `loom.yaml` sets `discussion: {reviewer_fan: ""}`.
     - `ValidateLocal` finds retired `discussion` covering the path. Its category is `none` (`internal/loomengine/config.go:680` `"discussion": none`), so the path is admitted with no template or category check.
     - The fan key is a Guard key, so a local file now bypasses that guard. Every `discussion.*`, `plan.*`, `parts.*`, `raddle.*` and `friction.*` key gets the same pass.
     - With a non-`none` category, the opposite happens: every new path under the area is refused.
   - The old-name refusal in D4 must tell an old bare key from the new map by shape, as D5 does for the move. Otherwise those five keys must stay out of the derived retired set. The record says neither.

2. **MEDIUM — D7: the board rewrite's place in the walk is unstated, and the obvious spot nests board writes.**
   - Today the hub-wide walk does its writes inside one `fabricengine.BoltWrite` closure run by `PullThenCommitWritten` (`reconcileHubWide` in `internal/hubreconcile/walk.go`). That is where the reinterpretations and their markers run.
   - D7 has the `run_settings` rewrite go "through `boardengine`'s own write path under its lock". That path pulls and commits the board itself. Called inside the hub-wide closure, it nests a board commit inside a board commit and risks a deadlock on the lock.
   - Called outside the closure, the marker and the rewritten `board.json` must land in the same commit, or a failure marks the hub done with the settings not carried.
   - The record needs to state where D7 runs and that its marker commits atomically with the rewrite.

3. **LOW — D4: refusing `--unset` of an old name takes away the manual fix for a leftover old key.**
   - Case: a pair skipped by the walk still holds `code_max_bounces` in a file. Its load is refused, naming `loom.code.max_bounces`.
   - `lym config loom --unset code_max_bounces` is refused too. That leaves only another reconcile run or a hand edit, which the way forward does not name.
   - `--unset` should admit old names, or the refusal should name the reconcile as the way forward.

4. **LOW — D5: an incomplete pair is never moved by this build's walk.**
   - `reconcileWorktree` skips a pair that is not complete and returns `skipped=false`. The build stamp is then written, so no later walk revisits that pair.
   - A pair that Seed-Child had already given a local `loom.yaml` with old names, then interrupted, keeps those names. Its next load is refused under D4.
   - Small window, but it misses D5's promise of "no operator step".

5. **LOW — D8: the recipe-reference parse may still allow only one dot.**
   - The exploration noted that `shedbuild`'s `operandPattern` check runs before the module.key split. A pattern that allows a single dot would reject `loom.split.reviewer_fan` at recipe build before D8's nested lookup runs.
   - D8 names the lookup but not this check.

**Per decision**

| Decision | Case walked | Result |
|---|---|---|
| D1 | `discussion` → `discussion.model`, `pr_review_ask_parent` → `pr.review_ask_parent`, `part_writer` → `parts.writer` | Table is consistent. Broken only through D4 (finding 1). |
| D2 | drift keeps `judge_ask_parent` | Met. |
| D3 | `reviewbudget`'s refusal of `RunSoftKey`, rendered by the helper as `loom.review.max_bounces` | Met. |
| D4 | local `discussion.reviewer_fan` | **Broken** (finding 1). The `--unset` case is finding 3. |
| D5 | hub base `code_auto_accept: false` | Met; see below. |
| D5 | local file holding old keys | Met, if the move runs before the prune as stated. The incomplete pair is finding 4. |
| D6 | untouched `profiles.yaml` | Met; see below. |
| D6 | operator-added flat profile | Met: the merge keeps it and the reinterpretation moves it. `configsync` runs no `CheckLoad`, so the not-yet-moved flat profile is not refused mid-walk. |
| D7 | entry with `{"loom":{"code_max_bounces":2}}` | Mechanism unclear (finding 2). |
| D8 | `FlagCodeAutoAccept` after the move | Met, since the move runs before the reinterpretation in the same closure. Recipe reference is finding 5. |
| D9 | categories keyed by the new dotted paths | Met, apart from the retired coverage in finding 1. |

- **D5, hub base:** `ReconcileHubWideAt` loops over every registered module's board file, the per-worktree loom module included, before `applyReinterpretations` in the same closure. So the file reads `code: {auto_accept: false}` before `FlagCodeAutoAccept` runs.
- **D6, untouched file:** the seed-only merge compares the file with the old flat template base. Whole profile entries that equal the base take the new template.

**Further calls**
- Read `walk.go` lines 199-280 (`reconcileHubWide`): my context lacked how the hub-wide write commits, which D7 depends on.
- Grepped `ReconcileHubWideAt`'s module loop and the `TopLevelValue` callers: my context lacked whether the hub-wide reconcile covers per-worktree modules' board files, and whether the "one caller" constraint is true. It is true.
- Grepped `configsync` and `hubreconcile` for `CheckLoad`, and `loomengine/config.go` for the bare keys' categories: my context lacked whether the reconcile load-checks `profiles.yaml`, which affects D6, and the category of `discussion`, which affects finding 1.
