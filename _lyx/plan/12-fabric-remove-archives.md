# Card 12 — fabric-remove-archives

**Edit:**
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#RemoveResult`

**Create:**
- `internal/fabricengine/remove_archive_integration_test.go`

**Prosa:**
- `CONSTRAINTS.md`
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/fabricengine#archiveWeftTip`
- `plan:internal/fabricengine#KindTagPushed`
- `internal/fabricengine#removeWeftWorktree`

**Intent:**
`Remove` archives the pair's weft tip before it tears anything down.

- `RemoveResult` gains `ArchiveTag string` (`json:"archive_tag,omitempty"`) and `ArchiveSkippedReason string` (`json:"archive_skipped_reason,omitempty"`).
- `Remove` calls `archiveWeftTip(rec, l, slug, weftBranch)` after its refusals (slug, prime, target existence, the two merge-in-progress checks) and before its first mutation, `removePortal`: a failed archive returns the error with the worktrees, portal, launchers and both branches still in place, so a plain re-run retries it.
- The archive runs whatever `remote` says, since it protects the local branch's commits as much as the remote copy, and `force` does not skip it: `force` answers dirtiness only, as it already does for a live merge record.
- A weft repo with no origin proceeds with the skip reason on the result.
- Update `Remove`'s doc comment with the archive step, its position, and its interaction with `force` and `remote`.

`Add`'s rollback is untouched: it calls `removeWeftWorktree` directly for a weft branch it just created, which carries no records, so it never archives.

Tests in `remove_archive_integration_test.go`, over a hubforge hub with a weft origin: after `Remove` an `archive/<slug>/…` tag on origin points at the weft tip that carried a committed `_lyx` file, and the result names it; a resumed `Remove` after a refusal on the same tip reuses the tag; an unreachable origin returns an error and leaves both worktrees, the portal, the launchers and both branches in place; `force: true` still archives; `remote: false` still pushes the tag; no origin skips with `ArchiveSkippedReason` set and completes the removal; and an `Add` rolled back after creating its weft branch leaves no `archive/` tag.
The fabricengine livestate and mutation-record tests that pin `Remove`'s recorded mutations gain the `tag_pushed` entry where their fixtures have an origin.

Docs: in `CONSTRAINTS.md`'s Fabric Git Invariant, add a bullet that every teardown of an existing pair's weft branch (`Remove`, `RemovePairBranch`, `Cleanup`) first pushes an `archive/<slug>/<tip>` tag to the weft origin, a rolled-back `Add` excepted, and that `force` never skips it.
In `internal/fabricengine/doc.go`, describe the archive step where `Remove`'s teardown order is documented.

**ImpactSummary:** Remove gains a network step ahead of any mutation and two result fields; lyx fabric remove and batten teardown now fail while a configured weft origin is unreachable.
