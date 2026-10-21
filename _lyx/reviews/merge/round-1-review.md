---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-pattern-fit
    location: PATTERN.md:13
    summary: The merged overview discards the wording the parent gave three entries it changed, hub-wide-config, hub-store-gc and friction-capture, so spawned agents no longer see the machine override, the maintenance half of the gc rule or the run-end reflection.
    fix: In PATTERN.md restore the parent's overview text of the three entries, taken from 860c5d03 — hub-wide-config "its file on the board only, its machine override beside it", hub-store-gc "Hub stores run no automatic gc or maintenance... only `fabricengine/housekeeping.go` sets that or runs `gc`", friction-capture "Go writes a note, reflected at the run's end, archived once covered". Then run the PATTERN headroom check the verify runs (`internal/patternlint`); if the overview grows past the line, cut the same number of bytes from entries that neither side changed in this merge and keep that detail in their background files, as `patternlint.WayForward` says.
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: PATTERN.md:44
    summary: The merge commit shortened the declared PATTERN-origin-reconcile entry to "sole judge of local branches", dropping "the" and "against origin" from the declared text.
    fix: In PATTERN.md line 44 end the entry with the declared words "the sole judge of local branches against origin." exactly as the decision record's PATTERN changes list them, and make up any headroom overage as F1's fix says.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/fabricengine/remove_localmode_integration_test.go:82
    summary: No test pins what a pair with a linked and populated root .scratch does under the task's local removal, snapshot and dirty gate, so the guarantee that the link and its content are neither dirty nor snapshotted rests on the shared exclude alone.
    fix: Extend the existing TestRemoveLocal (remove_localmode_integration_test.go:82) and the TestSnapshotUncommitted case that asserts the exact snapshot paths (same file, line 249) instead of adding a new test. Write a file under the pair's root .scratch before the call. Assert that the snapshot paths and the force dirty gate ignore the link and its content, and that the RemoveLocal record holds one link_removed at the root .scratch ahead of the warp worktree_removed.
---

### [MEDIUM] Merged overview drops the parent's wording of three entries

**Location:** PATTERN.md:13, PATTERN.md:39, PATTERN.md:78
**Issue:** The merge conflicted on PATTERN.md and was resolved toward the task's overview cut.
The result carries the task's text for three entries the parent itself reworded: `PATTERN-hub-wide-config` lost "its machine override beside it", `PATTERN-hub-store-gc` lost "no automatic gc or maintenance", and `PATTERN-friction-capture` lost "reflected at the run's end, archived once covered".
`git diff 860c5d03 HEAD -- PATTERN.md` shows each.
Spawned agents get only the overview inlined, so the machine-layer override of a hub-wide module and the run-end reflection no longer reach them, although the parent landed both and the decision record declares no change to these entries.
The background files keep the detail, so the loss is confined to the overview.
`PATTERN-config-layers` was reconciled correctly: the merge restored "machine layer included".
The Webster review's carried `pattern-entries-rewritten-undeclared` covers the task's own rewrites; this finding is the parent's text discarded at the merge.

### [LOW] Declared origin-reconcile entry shortened by the merge

**Location:** PATTERN.md:44
**Issue:** The task's tip carried the declared text "the sole judge of local branches against origin".
The merge commit's own change to PATTERN.md rewrote it to "sole judge of local branches", which the declared-changes rule judges against the listed text.
"Against origin" names what the judge compares with, so the shorter text is weaker than the declaration, though it still carries the invariant.

### [LOW] Linked .scratch has no test under the local removal, snapshot and dirty gate

**Location:** internal/fabricengine/remove_localmode_integration_test.go:82
**Issue:** The parent's root `.scratch` link meets three paths of the task: `RemoveLocal`, `SnapshotUncommitted` and the force dirty gate (`dirtyScopeAll` in `removeWarpWorktreeDir`).
Read as code they agree: the shared warp `info/exclude` carries `/.scratch` with no trailing slash, so `git status` lists neither link nor content, and the weft `.lyx` is excluded wholesale, so the link target's content is neither snapshotted nor dirty.
No test puts content under a pair's `.scratch` and asserts that, and no `RemoveLocal` test asserts the `link_removed` record.
The plain `Remove` test pins the unlink only for the other mode.
A regression to a trailing-slash exclude, or to a weft `.lyx` that is not excluded, would put scratch paths into a set-aside snapshot with nothing failing.

## Rejected

- Fork code-correctness LOW, `removePair` unlinks the root `.scratch` before the warp worktree directory removal, so a non-force dirty refusal leaves the pair without its link: the ordering is the parent's alone, matches how the junction sweep already precedes the same removal, and the pairteardown probe runs the refusals first; it is no interaction with the task.
- Fork code-tests LOW, the merged `runStaleBuildReconcile` might spawn a real detached board sync: `recordBoardSyncs` at `internal/hubreconcile/ensure_integration_test.go:321` replaces `spawnBoardSync` for that test, as the affected-set lens confirmed.
- Fork code-pattern-fit and code-affected-set claim `PATTERN-config-layers` lost "machine layer included": `PATTERN.md:21` of the merged tree carries it; only `PATTERN-hub-wide-config`, `PATTERN-hub-store-gc` and `PATTERN-friction-capture` were dropped, which F1 records.
- Fork code-pattern-fit's list of the task's other overview cuts (`PATTERN-hub-suffix`, `PATTERN-edit-directive`, `PATTERN-stencil-ownership`, `PATTERN-creel-store`, `PATTERN-batcher-registry`, `PATTERN-model-choice-mapping`, `PATTERN-testkit`, `PATTERN-push-both-sides`, `PATTERN-durable-vs-ephemeral-state`): the parent did not change these entries, so the cut lives in the task's range alone and the Webster review judged it.
- Fork code-correctness note that `cherryPickOnto` now reads any `ResetKeep` failure as `ErrReplayDirty`: it lives in the task's range alone.
- Test redundancy between a task test and a parent test: none found, the overlapping test files cover disjoint behaviours.
