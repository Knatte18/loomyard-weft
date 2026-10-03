# Card 1 — remove-refusal-probe-and-record-commit

**Edit:**
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#refuseDirtyWeftWorktree`
- `internal/fabricengine#ErrPairSiblingDirty`
- `internal/fabricengine/remove_siblingdirty_integration_test.go`
- `internal/fabricengine/remove_archive_integration_test.go`

**Create:**
- `plan:internal/fabricengine#Topology.RemoveRefusal` -> `func (t *Topology) RemoveRefusal(l *lyxcwd.Location, slug string, force bool) error`
- `plan:internal/fabricengine#commitPendingRecords` -> `func commitPendingRecords(rec *Mutations, l *lyxcwd.Location, slug, warpSHA string) (sha string, committed bool, err error)`
- `internal/fabricengine/remove_records_integration_test.go`

**Uses:**
- `internal/fabricengine#CommitWeftPaths`
- `internal/fabricengine#PathspecNames`
- `internal/fabricengine#archiveWeftTip`

**Intent:**
Implements D3 and D6 for the full path (task worktree present).

D3: move every refusal `Remove` raises before its first mutation into the exported read-only probe `Topology.RemoveRefusal`: slug validation, the prime refusal, both merge-in-progress directions, warp-side dirtiness without `force`, and sibling-side dirtiness outside the scoped pathspec without `force`.
`Remove` calls the same probe first, so the two can never drift.
The probe mutates nothing, takes no lock and pushes nothing.
Its name stays free of the fabric side vocabulary, because `internal/pairteardown` calls it.

D6: after the refusals and before `archiveWeftTip`, `Remove` commits the sibling worktree's uncommitted changes through `commitPendingRecords`.
It stages exactly the scoped pathspec `lyx fabric sync` uses, `ScopedPathspec(l.AnchorRel, PathspecNames(BoardDir(hub)))`, through `CommitWeftPaths`.
The message is the fixed `DefaultCommitMessage` with a `Warp-SHA` trailer carrying `warpSHA` (the task worktree's HEAD on this path), and no push.
The archive tag then covers the new tip.
The commit runs with or without `force`.
`KindCommitCreated` is recorded only when a commit landed (Mutation Record Invariant).

The old no-force refusal of sibling-side dirtiness inside the scoped pathspec goes.
`refuseDirtyWeftWorktree` keeps refusing, without `force`, only changes outside that pathspec, and its message keeps the substring "uncommitted changes" that the livestate matrix pins.
Because the dirtiness was never in the pathspec, its old "run lyx fabric sync" remedy is wrong; it names the out-of-pathspec paths and `--force` as the way past.
`ErrPairSiblingDirty`'s doc now says it marks content outside the record pathspec.
`force` still answers dirtiness only and never skips the commit or the archive.

Bound: the new commit stages only lyx-owned record paths in the pair's own sibling worktree and pushes nothing; it replaces a refusal whose own remedy was exactly this sync.

Tests:
- `remove_records_integration_test.go` covers an untracked stop report under the pair's `_lyx/shed/<slug>/drive-reports/` being committed and contained in the pushed archive tag.
  The same test covers out-of-pathspec sibling dirt refusing before any mutation (no tag, no commit, sibling branch tip unchanged) and the same dirt proceeding with `force`.
  It also checks that `RemoveRefusal` alone leaves the pair, its branches and the weft origin untouched for each refusal kind.
- Flip `TestRemove_UntrackedDriveReportRefusesWithSiblingDirty` to assert the report is committed and archived.
- Flip `TestRemove_WeftDirtyRefusalPushesNoTagAndResumeArchivesOnce` to assert a single Remove commits and archives the record.

**ImpactSummary:** Remove's refusal set and pre-archive step change; callers outside fabricengine see only the narrower ErrPairSiblingDirty meaning.

**Commit:** `1: fabric remove commits pending records and exposes its refusal probe`
