# Card 3 — reconcile-adopts-origin-weft

**Edit:**
- `internal/fabricengine#Topology.reconcileMissingWeft`
- `internal/fabricengine#ReconcileActionWeftRecreated`
- `internal/fabricengine/doc.go`
- `internal/fabriccli/adoptremote_integration_test.go`

**Uses:**
- `plan:internal/fabricengine#resolveWeftBranch`
- `internal/fabricengine#adoptWeftWorktree`
- `internal/fabricengine#Topology.repairPairWiring`
- `internal/fabricengine#ReconcileActionRawAdopted`

**Intent:**
D7's reconcile half: `reconcileMissingWeft` consults origin before its raw-warp and unmanaged branches.

After the weft-repo check, replace the `weftBranchExists` test with `resolveWeftBranch(rec, warpLayout, weftBranch, true)`.
- `exists` (local, or just created tracking origin): prune and `adoptWeftWorktree` as today, returning `ReconcileActionWeftRecreated`; the Detail says the branch existed locally, or was adopted from `origin`, so the existing wiring fall-through in `Topology.Reconcile` wires junctions, portal and launchers through `repairPairWiring`.
  This holds for the raw-warp case and the unmanaged case (managed warp worktree, no weft sibling) alike; for a raw warp worktree it replaces the dormant `raw_adopted` fork that path produced before.
- A resolver error sets `pr.Error` with the cause (it names `origin`) and returns `ReconcileActionWeftRecreated`, mirroring the existing adopt-failure branch; the error blocks the wiring fall-through, and no dormant weft is forked.
- `!exists`: the raw and unmanaged branches run unchanged.

Reconcile adopts an origin weft whether or not an `archive/<slug>/*` tag covers it.
Bound (D7): the archive-guard bypass only creates a local branch from origin and pushes, replaces or deletes nothing, and only for a slug whose warp worktree already exists locally on that branch, so no removed pair is revived without a local warp worktree; `add`'s replace-archived rule applies only where `add` itself creates the pair, so the two never decide the same slug.
For a raw warp worktree, which proves no live pair, the bound is weaker: a raw worktree on a branch reusing a removed pair's slug adopts that pair's archived weft and gets junctions wired into it; accepted because the content was already on origin, nothing is pushed or replaced, and the hub can remove the pair again.
A hub whose weft repo has no `origin` remote behaves as today.

Update `ReconcileActionWeftRecreated`'s comment (a branch that existed locally or on origin), the reconcile.go header, and `internal/fabricengine/doc.go`'s reconcile prose to say reconcile adopts an origin weft before forking a dormant one.

Tests, in `internal/fabriccli/adoptremote_integration_test.go` through `lyx fabric reconcile`:
- a raw warp worktree checked out on `<slug>` (created with `git worktree add` from the prime) with only `origin/<slug>-weft` present: the pair's `action` is `weft_recreated`, the local weft branch tracks `origin/<slug>-weft`, `.weft/` exists, and the pair's junctions, portal and launchers are wired;
  the same with an `archive/<slug>/*` tag covering the origin tip still adopts;
  with the weft repo's `origin` URL pointed at a missing path, the pair reports an `error` and no `<slug>-weft` branch exists locally;
- a managed pair whose weft worktree and local weft branch were removed (origin keeps `<slug>-weft`): `action` is `weft_recreated`, the weft tracks origin, and the wiring is repaired;
- with the weft repo's `origin` remote removed, a raw warp worktree still gets today's dormant `raw_adopted` fork.

**ImpactSummary:** Reconcile's missing-weft branch adopts an origin weft before the raw/unmanaged branches; raw worktrees with an origin weft now end wired as weft_recreated instead of dormant raw_adopted.

**Commit:** 3: fabric: reconcile adopts a pair's weft branch from origin before forking
