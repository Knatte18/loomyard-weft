# fabric: lock dir at birth, adopt origin branches, report config diffs

Three fabric bugs, each with a regression test that drives the real `lyx fabric` or loom wiring on a `hubforge` fixture with a bare origin.

## What changes

**A weft worktree always has `.weft/`.**
Every path in `internal/fabricengine` that runs a weft `git worktree add` now creates the lock directory in the same mutation, so a strand in a recreated pair no longer loses `LYX_PARENT` silently.
The paths are fork, drop-runs fork, `add`'s adopt, `reconcile`'s adopt and the new origin adopt.
`lyx fabric reconcile` also creates a missing `.weft/` in an otherwise healthy pair and names the repair in that pair's `Detail`; it adds no new action value.
`ReadOriginFor` still answers "no record" when the weft worktree or the record file is absent.
When the record exists and `.weft/` does not, it now returns an error that names the worktree and points to `lyx fabric reconcile`.
`hubgeom.ResolveParent` propagates that error.

**Fabric adopts a pair's branches from origin instead of forking past them.**
A new resolver in `internal/fabricengine/weftbranch.go` finds a pair's weft branch: local first, then `origin/<slug>-weft` (fetched into a tracking branch), else none.
`add`, `reconcile` and `checkout` all use it.

- `reconcile` adopts an origin weft as `weft_recreated` for both the raw-warp and the unmanaged cases, so junctions, portal and launchers are wired.
- `checkout` switches to a local branch tracking the origin weft.
- `add` treats a pair as live when the weft exists locally, or exists on origin without an `archive/<slug>/*` tag covering its tip.
  For a live pair it adopts the weft and builds the warp branch tracking `origin/<slug>`, where it used to refuse a diverged leftover.
  A local weft that is a strict ancestor of origin's is fast-forwarded first; a true divergence still refuses.
  An adopted weft that already has an origin record keeps it, so a stacked task moved between machines keeps its parent.
- `add` with `SkipGit` or `SkipPush` never consults origin and forks as before.
- Without an `origin` remote configured, `reconcile` and `checkout` fork as before.

**Landing reports a task's config changes to the hub.**
Finalize reads the task's changes to its per-worktree `_lyx/config/` files (modules not marked `HubWide`) between the weft merge-base and the task's weft tip.
At Done it queues one orch notice through `orchengine.QueueNotice` naming the slug, the files and both SHAs, so the hub can re-apply each change meant for the parent.
`ReadConfigChanges` in `internal/fabricengine/configchanges.go` computes the diff, and `internal/loomcli` hands it to `landingshed` as a told function.
Finalize never writes the parent's config.

Closes: none; the work is tracked by board entry `bugfix-fabric`.

## Design decisions to check

- **Notice, not carry.**
  Per-pair tuning is legitimate, such as a small task's `review_max_bounces` in the pair's `loom.yaml`, so an automatic carry would push per-pair overrides into the parent.
  The hub is the only party that knows which change was meant for the parent.
- **The notice never stops the landing.**
  A failed diff read lands as usual and queues one notice saying the diff could not be read.
  A `QueueNotice` error is a logged warning.
  `queued=false` for want of an orch strand is accepted as is.
  A Finalize that reaches Done again after a resume queues the notice again.
- **Origin errors stop the pair or verb.**
  The resolver treats a failed origin probe or fetch as an error, never as a fall-through to forking.
  `reconcile` and `checkout` have no offline opt-out, so with origin unreachable and no local weft branch they now fail naming origin where they forked before.
- **`reconcile` and `checkout` adopt an origin weft even when an archive tag covers it.**
  Nothing on origin is pushed or replaced, and `reconcile` needs a local warp worktree on that slug.
  A raw warp worktree reusing a removed pair's slug can adopt that pair's archived weft; this is accepted because the content was already on origin.
- **Unarchived remote weft now makes `add` adopt.**
  The `leftoverUnarchivedWeft` refusal is retired.
  A removed pair whose archive step failed comes back with its stale run records and warp branch, and a plain `lyx fabric remove` archives it again.
  A removed pair with an archived weft and no local weft still hits today's refusals.
- **Rollback.**
  It deletes only local branches this `add` created, an origin-tracked one included, through the existing `deleteBranch` gate.
  It never touches origin, and it does not rewind a fast-forwarded local weft.
- **Reviewer resolution.**
  `loomcli`'s reviewer resolution turns from "no reviewer" into an error for the lock-directory-missing case; `reconcile` clears it and the creation changes stop it arising.
- **Docs.**
  The changed behavior is documented in the `doc.go` of `fabricengine`, `hubgeom` and `landingshed`.

## To check by hand

- Four `PATTERN-comment-line-breaks` nits in Go comments remain open and are formatting-only; no code defect was found.
  They are at `internal/fabricengine/checkout.go` near line 137, `internal/fabricengine/remoteleftover.go` near lines 2 and 20, and `internal/landingshed/deps.go` near lines 126 and 138.
  Sweep them before or at landing.
- `internal/testkit/envkit/envkit.go` gained the two new nil-is-absent landing seams in its nil-legal list; this is the one change outside the planned files.
- Moving a real task between two machines through `lyx fabric add` was not run by hand; the tests cover it on a bare-origin fixture.
