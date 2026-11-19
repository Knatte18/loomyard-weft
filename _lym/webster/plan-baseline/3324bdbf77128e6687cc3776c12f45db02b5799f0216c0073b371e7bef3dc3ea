# Card 2 — records-paths-and-commit-predicates

**Create:**
- `internal/fabricengine#MergeTrailerKey`
- `internal/fabricengine#AnchorChangeTrailerKey`
- `internal/fabricengine#RecordsPathsAt`
- `internal/fabricengine#IsRecordsOnlyCommit`
- `internal/fabricengine#IsFabricMergeCommit`
- `internal/fabricengine_test#TestRecordsPathsAt`
- `internal/fabricengine_test#TestCommitPredicates`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#committedMarker`
- `internal/fabricengine#trailerValue`

**Intent:** The git reads D5 and D7 define, which later cards build on. All are reads; none writes a ref or the worktree.

`RecordsPathsAt` returns the records paths of the commit at `rev`:
the union of the anchors the `.lym-anchor` at `rev` and at its first parent name (read through `committedMarker`; a root commit has no parent; an absent marker contributes no anchor),
each joined as `<anchor>/_lym` through `ScopedPathspec` with `lymdirs.LymDirName`, plus `lymcwd.AnchorFileName`.
The list is sorted and deduplicated, forward-slashed. A merge side's records paths are `RecordsPathsAt` of the side's tip.

`IsRecordsOnlyCommit` is true for a commit with exactly one parent (or none) whose every changed path, from `git diff-tree -r --name-only --no-commit-id` against the parent or the empty tree, equals a records path or lies under a `<anchor>/_lym/` one.
Its message is never read, so a commit changing a code path is never records-only whatever trailer it carries.

`IsFabricMergeCommit` is true for a two-parent commit whose message carries a `MergeTrailerKey` trailer (read through `trailerValue`)
and whose tree equals the first parent's records paths laid over the clean merge of the two parents' records-stripped trees.
It builds each stripped tree in a temporary index under `lymtemp` (`GIT_INDEX_FILE`, `read-tree`, `rm --cached -r --ignore-unmatch` of the union of both parents' records paths, `write-tree`), as `worktreetree.go` builds one,
merges the stripped parents over their stripped merge base with `git merge-tree --write-tree --merge-base`, and answers false on a conflicted merge.
It then lays the first parent's entries under the records paths over the result (`ls-tree -r` into `update-index --index-info` on the result's index) and compares the written tree to the commit's.

`MergeTrailerKey` is the trailer every fabric merge commit carries (card 9); `AnchorChangeTrailerKey` the one fabric's anchor-change commit carries (card 8).

Tests (tier1, hermetic repositories through `gitkit`):
- `TestRecordsPathsAt` rows: `RootAnchor` (`.` gives `_lym` and `.lym-anchor`); `SubpathAnchor` (`backend/_lym`); `AnchorChangedInTheCommit` (parent `.`, commit `backend`, gives both); `NoMarker` (only `.lym-anchor`); `RootCommit`.
- `TestCommitPredicates` table over one repository, each row naming the commit and both wanted answers:
  `RecordsOnlyEdit` (records-only true); `MarkerEditOnly` (true); `CodeEdit` (false); `CodeEditCarryingTheMergeTrailer` (records-only false, fabric merge false);
  `MergeWithTrailerAndReceiverRecords` (fabric merge true); `MergeWithoutTrailer` (false); `MergeCarryingSourceRecords` (false);
  `MergeWithHandResolvedCodeConflict` (false); `SquashOfATask` (one parent changing code: both false);
  `RootCommitOfRecordsOnly` (a parentless commit holding only records paths: records-only true, read against the empty tree).

**ImpactSummary:** Additive only: new exports in fabricengine and a doc paragraph; no existing caller changes.
