# Card 6 — board-and-archive-readers

**Create:**
- `internal/boardengine#BoardFileRel`
- `internal/boardengine#ParseBoardTasks`
- `internal/fabricengine#ArchiveTagPrefix`
- `internal/fabricengine#ArchiveTagSlug`
- `internal/fabricengine#TaskCommitOf`
- `//internal/fabricengine/archivetag_test.go`

**Edit:**
- `internal/fabricengine#archiveWeftTip`
- `internal/fabricengine#remoteArchiveTags`
- `internal/fabricengine#hasWeftEvidence`
- `internal/boardengine_test#TestLoad`
- `internal/fabricengine#TestParseWarpSHATrailer`
- `internal/fabricengine_test#TestArchiveWeftTip`
- `//cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D10 reads board labels and done marks from board history, finds each archived run by its archive tag, and joins a run to its landing through the task commit its records commit names.
Each format keeps its owner, and the names the report calls carry no fabric side token, since `PATTERN-fabric-vocabulary` bars one from a non-owner file even when an owner exports it.

`boardengine.BoardFileRel` (in `store.go`) returns the board file's name relative to the board directory, which is the board checkout's root, the `boardFile` constant.
`boardengine.ParseBoardTasks` decodes `board.json` bytes read from a board commit: it unmarshals the stored file the way `Store.Load` does, refuses a version other than `storeVersion` with the error `Store.Load` gives, and returns `migrateEntries(file.Entries, nil)`, so an entry in the old tier-and-type shape still yields its labels and an unknown key is ignored.
It takes no lock and reads no legacy file; `Store.Load` is unchanged.

`fabricengine.ArchiveTagPrefix` (in `archive.go`) is the short-name prefix every archive tag carries, and `archiveWeftTip`, `remoteArchiveTags` and `hasWeftEvidence` compose their tag names and `refs/tags/` patterns from it instead of their own `"archive/"` literals, with the same bytes, so the namespace has one spelling.
`fabricengine.ArchiveTagSlug` returns the slug of a short tag name `archive/<slug>/<sha>`: true only for that prefix, a non-empty slug segment and a non-empty final segment.
`fabricengine.TaskCommitOf` (in `trailer.go`) returns the code-side commit a records commit message names in its trailer, delegating to `parseWarpSHATrailer`, so the report reads the trailer without spelling the trailer key.

Tests:
- `TestLoad` gains a check in its runner for every row whose `body` is non-empty: with `wantErr` empty, `ParseBoardTasks([]byte(body))` returns entries whose slugs and labels equal the loaded store's, which covers the old-shape migration rows; with `wantErr` set, `ParseBoardTasks` returns an error too, which the existing "an unknown version is refused naming it" and "a corrupt file is an error" rows reach.
- `TestParseWarpSHATrailer` asserts `TaskCommitOf(message)` equals `parseWarpSHATrailer(message)` for every row.
- `TestArchiveTagSlug` (new, untagged, package `fabricengine`, in `archivetag_test.go`) is a table: `archive/my-slug/0123456789ab` returns `my-slug`, true, and `archive/x`, `archive//abc`, `archive/x/` and `snapshot/x/abc` return false.
- `TestArchiveWeftTip` asserts `ArchiveTagSlug` of the tag it pushed returns the test's slug, which pins the prefix to the writer.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/fabricengine` to 645.

**ImpactSummary:** `archiveWeftTip`, `remoteArchiveTags` and `hasWeftEvidence` keep the exact names and patterns they use, so their callers and the archive and leftover tests are unaffected; the other changes are new members and test assertions.

**Commit:** `6: board, archive tag and task commit readers`
