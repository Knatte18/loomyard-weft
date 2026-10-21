# Fixer report — discussion round 1

All eleven findings were fixed in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged.

## Changes

- **F1** (fixed as proposed): D3 now says the stale-junction sweep never claims the root `.scratch` link and never strips its `/.scratch` exclude line.
  A criterion now requires a second reconcile on a converged pair to keep the link and the exclude line and to report no stale removal.
  The self-check placed the skip in `applyStaleRemoval` and `Unwire`, not in the shared `scanOnDiskJunctionNames` (see Self-check).
- **F2** (fixed as proposed, unconditional variant): D4 writes `/.scratch` unconditionally with no `check-ignore` probe, and explains why the trailing-slash probe answers wrongly given the tracked `**/.scratch/` line.
  The exclude criterion's fixture now carries a tracked `.gitignore` that holds `**/.scratch/`.
- **F3** (fixed as proposed): D1 now writes the exclude first, and a failed write creates no link, renames nothing and is reported.
  D2's Bound now says a failed exclude leaves the worktree as it was before the step, and a criterion was added for it.
- **F4** (fixed as proposed): the post-rename refused link is now its own D1 case 5, which reports the target path and the way forward.
  D2's Bound covers case 5's reachability.
  Criterion 5 is now scoped to case 4, and a case 5 criterion was added.
- **F5** (fixed as proposed): D1 now says `WireJunctionsWith` runs the step only on add, checkout and clone, and that reconcile's re-wire calls a form that skips it.
  D3 now says `repairPairWiring` runs the step once on both branches, after the junction repair whether or not it failed, and that the step returns its outcome for the detail.
  D1 now builds the target from the weft root, not the warp `.lyx` link, so a failed repair does not move it.
- **F6** (fixed as proposed): criterion 7 was reworded with the "when the step would run" qualifier, and now includes the recreated-weft case.
- **F7** (fixed as proposed): D1 case 2 classifies by comparing the target with `fslink.RawTarget`.
  The Notes bullet no longer names `seedLyxJunction`'s resolving verify branch as the model.
- **F8** (fixed as proposed): D1's Rationale now says scratch lives and dies with the pair's weft worktree, including a recreated one.
- **F9** (fixed as proposed): criteria were added for mutation recording on conversion and for unwire leaving the link and the exclude line.
- **F10** (fixed as proposed, new-kind variant): D2 names a new `path_renamed` kind, its Target and Detail, and its guard-test entry.
  It also names `link_created` and `file_written` for the link and the exclude.
- **F11** (fixed as proposed): a new D6 has the run append the junction modification-time check to `fabric-windows` through `lyx board`, and the Open risks bullet now points at it.

## Self-check

- fix-consistency, MEDIUM: D3's "raw worktree whose junctions this pass has not wired" clashed with running the step after a failed repair. Fixed as proposed: it now reads "a raw-adopted worktree, which never reaches `repairPairWiring`".
- fix-consistency, LOW: D2's reporting sentence left out D1 case 5. Fixed as proposed: case 5 is now named.
- fix-consistency, LOW: D2's Rationale "lose content" conflicted with case 5. Fixed with own change: it now reads "delete content", which case 5 never does, instead of loosening the Bound.
- fix-claims, MEDIUM: the target build left out `AnchorRel` (`junction.go:61`). Fixed as proposed: the target is now weft root, `AnchorRel`, `DotLyxDirName`, as `WarpJunctions` builds it.
- fix-way-forward, LOW: the exclude-failure report had no way forward. Fixed as proposed: the report names the exclude path and the error, and the way forward is to clear what blocks the write and rerun reconcile; the criterion was updated.
- fix-proposal, MEDIUM: putting the skip in `scanOnDiskJunctionNames` would stop `lyx fabric remove` unlinking `.scratch` before the directory removal (`remove.go:250`). Fixed with own change: the skip now lives in `applyStaleRemoval` and `Unwire` (`unwire.go:72`, which needs it for D3's unwire rule), and D3 states that remove still unlinks the link as `link_removed`. The Notes bullet was updated to match.

## Disputed

None.

## Deferred

None.
