# Decision record — fabric remove pushes the archive tag before its dirty checks

## Goal

`Topology.Remove` (`internal/fabricengine/remove.go`) must not push an `archive/<slug>/<tip>` tag, or tear anything down, when its no-force dirtiness checks are going to refuse.
Today a teardown refused for a dirty sibling has already pushed a tag that misses the uncommitted records,
and the resume after `lyx loom commit-records` pushes a second tag at the new tip.
After this task a refused `Remove` leaves the hub and the weft origin exactly as it found them, and the successful resume pushes the pair's only archive tag.

## Scope

In:

- Reorder `Remove` so every refusal, the no-force dirtiness checks included, precedes the archive step, and the archive precedes every removal.
- Delete the stranded-teardown remedy machinery the reorder makes dead (`nameStrandedPortalTeardown` and the `archivedEntries` bookkeeping).
- Rewrite the `Remove` tests that pin the old order.
- Update the comments and package doc that describe the old order.

Out:

- `Cleanup` and `RemovePairBranch` (`cleanup.go`): each already refuses before it archives and has no dirtiness check after its archive step, so neither carries this defect.
- `archiveWeftTip` itself (`archive.go`): its behaviour is unchanged.
- `battencli`'s `teardownRefusal`: its remedies stay correct under the new order, so its wording does not change.
- Pruning or deduplicating archive tags already pushed by the old order.

## Decisions

- **D1 — New `Remove` order.**
  slug validation → prime refusal → target-exists check → origin-record read → merge-in-progress refusals → no-force warp dirtiness check → no-force weft dirtiness check (`refuseDirtyWeftWorktree`) → `archiveWeftTip` → portal and launcher teardown → junction sweep → warp worktree removal → weft teardown → warp branch deletion.
  Rationale: the board task asks for the archive after the dirty checks and still before any removal.
  Portal and launcher teardown is a removal that today sits before the dirty checks, so the only order meeting both conditions moves the dirty checks ahead of the portal and launcher teardown.

- **D2 — The portal and launcher teardown moves after the dirty checks; its old position has no remaining ground.**
  Rationale: the file header justifies tearing the portal down before the dirty checks "so the teardown still runs when the worktree dir itself is already gone".
  That rationale dates from `9de0c46d6`, where the portal teardown ran before the target-exists check.
  The teardown has since moved after that check, and a missing worktree dir now returns "worktree not found" before any teardown, so the case the justification names never reaches the dirty checks.
  Under `force` the dirty checks are skipped, so forced teardowns still reach the portal and launcher teardown unconditionally.
  A worktree dir that exists but is not a git checkout refuses at the status probe either way; the new order only stops it stranding the portal first.

- **D3 — Delete `nameStrandedPortalTeardown` and the `archivedEntries` count.**
  The dirty refusals return their errors unwrapped by any remedy.
  Rationale: with nothing recorded before the dirty checks, the helper's "something was stranded" branch can never fire; keeping it is dead code.
  The refusal errors themselves keep their current text and, for the weft side, their `ErrPairSiblingDirty` wrapping.

- **D4 — `force` semantics are unchanged.**
  `force` still skips only the dirtiness checks and never the archive.
  Rationale: `force` answers dirtiness only (the existing documented rule and `TestRemove_ForceStillArchives`).

- **D5 — A failed status probe refuses before the archive.**
  `check warp worktree status` and `check weft worktree status` errors now return with no tag pushed and nothing torn down.
  Rationale: the probe failure is a refusal like the dirty verdict; it belongs with the other refusals.

- **D6 — Test changes in `internal/fabricengine`.**
  - `TestRemove_ResumedRemoveReusesArchiveTag` is replaced by two tests:
    one asserting a weft-dirty no-force `Remove` refuses with `ErrPairSiblingDirty` and pushes no tag, then that committing the dirt and re-running pushes exactly one `archive/<slug>/` tag, at the post-commit tip;
    one asserting a same-tip tag already on origin and in the weft repo (the state an archive leaves when a later teardown step failed) is reused, with `ArchiveTag` naming it.
  - `TestRemove_RefusalNamesStrandedPortalTeardown` is rewritten to assert the opposite outcome: a no-force dirty refusal leaves the portal, the launchers, both worktrees and both branches in place, records no mutation, and names no reconcile remedy.
  - `TestRemove_RefusalWithNothingStrandedOmitsRemedy` is deleted; the rewritten refusal test subsumes it.
  - `TestRemove_StatusFailureNamesPathAndCommandOnce` stays unchanged.
  Rationale: the reuse path in `archiveWeftTip` has no other test (`archive_integration_test.go` does not cover it), so it keeps one; the remaining tests pinned the old order.

- **D7 — Documentation updates land in the same commit.**
  `remove.go`'s file header and `Remove` doc comment, `nameStrandedPortalTeardown`'s removal, the `internal/fabricengine/doc.go` passage on the archive step, and the file headers of `remove_archive_integration_test.go` and `remove_refusal_remedy_integration_test.go` describe the new order.
  `CONSTRAINTS.md` ("first pushes an `archive/<slug>/<tip>` tag") and `docs/overview.md` stay true and do not change.
  Rationale: CLAUDE.md's documentation lifecycle; no new cross-cutting invariant is introduced.

## Constraints

- `CONSTRAINTS.md`: every teardown of an existing pair's weft branch still pushes an archive tag before deleting that branch, and `force` never skips it.
- A failed archive must still leave the worktrees, portal, launchers and both branches in place, so a plain re-run retries it.
- `ErrPairSiblingDirty` and `ErrArchiveFailed` remain the sentinels `battencli` matches with `errors.Is`.
- Refusal message texts stay byte-identical: the livestate refusal matrix compares `Remove`'s dirty-worktree message against `destroy.go`'s.
- Comments and markdown follow the repo's semantic-line-break rule.

## Auto-mode assumptions

- The board task's "still before any removal" covers the portal and launcher teardown, so the dirty checks move ahead of it (D1, D2).
- Deleting the dead remedy helper is part of this task rather than a follow-up, because leaving it would ship dead code (D3).
- Duplicate archive tags already on remotes from the old order are left alone.

## Open risks

- The livestate matrix tests (`livestate_*_test.go`) and the mutation oracle may encode the old per-refusal mutation records for `Remove`; the full integration suite must run to surface any that do.
- `internal/battencli`'s lifecycle integration tests exercise teardown refusals end to end; one may assert a tag exists after a refused teardown.

## Acceptance criteria

- A no-force `Remove` of a pair with uncommitted weft changes returns an error matching `ErrPairSiblingDirty`, and afterwards: no `archive/<slug>/` tag exists on the weft origin or in the local weft repo; the portal, launchers, both worktrees and both branches exist; the result's mutation record is empty.
- The same holds for a no-force `Remove` refused for uncommitted warp changes, and the error does not mention `lyx fabric reconcile`.
- Committing the weft changes and re-running `Remove` succeeds and leaves exactly one `archive/<slug>/` tag on the weft origin, pointing at the post-commit weft tip.
- A same-tip archive tag already present locally and on origin is reused: `Remove` succeeds and `ArchiveTag` names that tag.
- A no-force `Remove` whose warp status probe fails refuses with no archive tag pushed.
- `force=true` against a dirty pair still pushes the archive tag and completes (existing `TestRemove_ForceStillArchives`).
- An unreachable weft origin still fails closed with everything in place (existing `TestRemove_UnreachableOriginFailsClosed`).
- `grep -rn nameStrandedPortalTeardown` over the repo returns nothing.
- `go test ./...` and `go test -tags integration ./internal/fabricengine/... ./internal/battencli/...` pass.

## Notes for the plan writer

- The change is confined to `internal/fabricengine/remove.go`, its doc passage in `doc.go`, and the `remove_*_integration_test.go` files named in D6.
- The weft dirtiness check handles an absent weft worktree as clean, so the reorder does not change the half-present-pair path.
- `battencli`'s `teardownRefusal` doc says a failed archive "runs before any teardown mutation"; that stays true and needs no edit.
