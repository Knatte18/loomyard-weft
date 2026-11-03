# Card 3 — add-drops-stale-tracking

**Edit:**
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.add`
- `internal/fabricengine#weftLeftover`
- `internal/fabricengine#probeWeftLeftover`
- `internal/fabricengine#probeWarpLeftover` -> `func probeWarpLeftover(l *lymcwd.Location, slug, warpBranch string, live bool) (adoptTip string, onOrigin bool, err error)`

**Create:**
- `plan:internal/fabricengine_test#TestAdd_DropsStaleTrackingRef` -> `func TestAdd_DropsStaleTrackingRef(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/gitrepo#Repo.DropRemoteTracking`
- `internal/fabricengine_test#removedPair`
- `internal/fabricengine_test#pushNewPair`
- `internal/fabricengine_test#requireNothingCreated`

**Intent:** Decision record D8: `Add` drops a former pair's remote-tracking ref and upstream config for a branch origin lacks.

- `weftLeftover` gains a field `originAbsent bool`, set by `probeWeftLeftover` when `remoteHeadTip` finds no weft branch on origin (the `tip == ""` return, with or without a local branch).
- `probeWarpLeftover` also returns whether origin holds the warp branch (`onOrigin`), true whenever `remoteHeadTip` found a tip, whatever the adopt answer; its only caller is `Topology.add`.
- In `Topology.add`, once every pre-flight refusal has passed and before the first worktree is made (beside `disableStoreAutoGC`), for each side whose probe ran and read the branch absent on origin — a weft branch forked or adopted locally with `originAbsent`, a warp branch forked with `onOrigin` false — it calls `gitrepo.Repo.DropRemoteTracking(origin, branch)` in that side's repository (`RecordsRepoRoot` for the weft, the acting worktree for the warp).
  A failed drop fails the Add before any worktree is made, naming the branch and the cause.
  Under `SkipGit` or `SkipPush` no probe runs and nothing is dropped.
  A local warp branch being adopted skips the warp probe today, so it is not dropped here; card 4 runs that probe under its adopt-only option.
- Bound: the dropped ref names a branch origin does not hold, so no reconcile loses an origin head; a rollback restores neither the ref nor the upstream config.
  Step 11's replace of an archived leftover already drops its ref through its own delete push and is unchanged.
- Add the drop and its bound to `Topology.Add`'s doc comment and to `internal/fabricengine/doc.go`'s `Add` pre-flight paragraph.

Test `TestAdd_DropsStaleTrackingRef`, `tier1`, one hub per row (the `add_remoteleftover_integration_test.go` fixtures):
- Run A's pair is pushed (`pushNewPair`) and removed so its weft branch is deleted on origin while the records repository keeps the weft branch's remote-tracking ref and its upstream merge config (the test's fixture re-creates both after the removal when the removal dropped them), then `Add` of the same slug forks a new pair: the records repository has no tracking ref and no upstream config for the weft branch, and a `ReconcileOrigin` run scope with `JudgeOnly` before any push of the new pair reports no `BranchGone` outcome for the pair's branches.
- The same with a stale warp remote-tracking ref and upstream merge config: both are gone after `Add`.
- A `config.lock` file planted in the records repository's git dir before `Add`, with a stale weft tracking ref standing and `branch.<weft>.remote` and `branch.<weft>.merge` set on the records repository's weft branch before the lock is planted, so the drop's config write fails: `Add` returns an error naming the weft branch, and `requireNothingCreated` holds.

**ImpactSummary:** Touches only `Add`'s pre-flight, its doc and its two probe helpers, whose sole caller is `Topology.add`; every caller of `Add` sees one more pre-mutation step and no change to an existing refusal text.

**Commit:** `3: Add drops a former pair's tracking ref for a branch origin lacks`
