# fabricengine: Remove archives only after its dirtiness checks pass

`Topology.Remove` used to push the `archive/<slug>/<tip>` tag before its no-force dirtiness checks.
A teardown refused for a dirty sibling had already pushed a tag that missed the uncommitted records,
and the resume after `lyx loom commit-records` pushed a second tag at the new tip.

Now every refusal runs before the archive, and the archive runs before every removal.
The order is: slug validation, prime refusal, target-exists, origin-record read, merge-in-progress refusals, warp dirtiness check, weft dirtiness check, `archiveWeftTip`, portal and launcher teardown, junction sweep, worktree removals, branch deletions.
A refused `Remove` leaves the hub and the weft origin as it found them, and the successful resume pushes the pair's only archive tag.
A gate refusal or a failed teardown step after the archive still leaves the tag, which the re-run reuses.

## Design decisions

- The dirtiness checks moved ahead of the portal and launcher teardown, because that teardown is a removal and the archive must precede it.
  The old justification for tearing the portal down first, that it must still run when the worktree dir is gone, no longer applies: a missing dir already returns "worktree not found" at the target-exists check.
- A failed `check warp worktree status` or `check weft worktree status` probe is a refusal and now returns before the archive.
- `force` is unchanged: it skips only the dirtiness checks and never the archive.
- `nameStrandedPortalTeardown` and the `archivedEntries` bookkeeping are deleted.
  With nothing torn down before the refusals, the reconcile remedy can never apply.
  The refusal texts and the `ErrPairSiblingDirty` and `ErrArchiveFailed` sentinels are unchanged, so `battencli` needs no edit.
- `Cleanup` and `RemovePairBranch` already refuse before archiving and are untouched.
  Archive tags already pushed by the old order are not pruned.
- `CONSTRAINTS.md` and `docs/overview.md` stay true as written.
  The `remove.go` header and doc comment, the `doc.go` passage, and the test file headers describe the new order.

## Tests

The tests that pinned the old order were rewritten:

- A weft-dirty no-force `Remove` refuses with `ErrPairSiblingDirty` and pushes no tag; after committing the dirt, the re-run pushes exactly one tag at the post-commit tip.
- A same-tip tag already on origin and in the weft repo is reused, and `ArchiveTag` names it.
- A dirty refusal leaves the portal, launchers, both worktrees and both branches in place, records no mutation, and names no reconcile remedy.
- The `removeCase` livestate cells and the mutation-record refusal test assert the new outcome.

## To check by hand

Nothing outstanding: the build, `go test ./...` and the integration suite passed.
