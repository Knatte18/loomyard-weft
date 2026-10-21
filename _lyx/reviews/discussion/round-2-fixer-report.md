# Round 2 fixer-report — discussion

## Changes

- F1 (MEDIUM), fixed differently: D3 now says `lyx fabric remove` unlinks the root `.scratch` link itself, by its root path and independent of `scanOnDiskJunctionNames`, through `destroy.go`'s `removeLink`, recorded as `link_removed`, before the warp worktree directory removal.
  Beyond the proposal, the unlink runs only when `linkIsFabricOwned` claims the link, and at a root anchor the scratch name is dropped from the scanned names so the link is unlinked once, since the scan does return it there (`internal/fabricengine/reconcile.go:725`, `.scratch` is not in `HubReservedNames`).
  The proposed sentence "the stale-removal and unwire skip applies to that same root entry" was replaced by an anchor-scoped statement, because the name-drop acts on `<worktree>/<AnchorRel>` and reaches the root entry only at a root anchor.
  D3's Bound now names remove's unlink as the one other verb that touches `.scratch`, Scope gains a remove bullet, and the plan-writer notes name `Remove`'s junction sweep as the join point.
  Added the acceptance criterion as proposed.
- F2 (MEDIUM), fixed as proposed: added D1 case 6 (failed target `MkdirAll`, refused case-1 link), D2's report list and Bound now cover it, and an acceptance criterion pins it.
  The Bound says what the next pass classifies after a refused case-1 link, by what then holds the root `.scratch`.
- F3 (MEDIUM), fixed as proposed: case 3 requires no tracked path under the root `.scratch`, checked through `PathTracked` (`internal/fabricengine/pathtracked.go:15`).
  Case 4 gains a tracked shape with its own way forward (stop tracking `.scratch` first), because the generic way forward would delete tracked paths.
  D2's Bound states the rename never moves tracked content, an acceptance criterion pins the tracked fixture, and support-log Q3 carries the new condition.
- F4 (LOW), fixed as proposed: D1 states the step replaces `seedScratchExclude`'s call, `seedScratchExclude` is deleted, and `ExcludeAnchoredDir` stays for its other callers (`internal/ideengine/spawn.go:102`).
- F5 (LOW), fixed as proposed: D2's `path_renamed` sentence now says it lands in the same commit as its recording site.
  The plan-writer notes name the real slot a new Kind needs: `manifestObservableKind` in `internal/fabricengine/livestate_mutationoracle_test.go`.

## Self-check

- fix-consistency, MEDIUM: D3's "skip applies to that same root entry" contradicted the scan-based skip on a subpath anchor. Fixed as proposed: the skip is scoped to a root anchor and the sentence removed.
- fix-consistency, LOW: Scope "In" lacked the remove-side unlink. Fixed as proposed.
- fix-consistency, LOW: support-log Q3 lacked the tracked-content condition. Fixed as proposed.
- fix-claims: no findings.
- fix-way-forward, MEDIUM: a failed tracked-path check had the untrack way forward and no error in its report. Fixed as proposed: it moved to case 6, whose report carries the error.
- fix-way-forward, LOW, merged with fix-proposal LOW 3: the Bound claimed the next pass after a refused case-1 link is always a case-4 leave. Fixed as proposed: the Bound names case 4, case 2 or case 1 by what holds the root.
- fix-way-forward, LOW: case 6's "clear what blocks the root `.scratch`" could discard agent content. Fixed as proposed: it now says to move content into the target before removing.
- fix-proposal, LOW: the F5 replacement named no slot, though the mutation oracle requires a classification row and Detail-subtree coverage for `path_renamed`. Fixed as proposed, in the plan-writer notes.
- fix-proposal, LOW: a tracked `.scratch` file or link still got the generic "remove the root `.scratch`" way forward. Fixed as proposed: the tracked leave shape covers any tracked `.scratch`, whatever its shape.

## Disputed

None.

## Deferred

None.
