# Card 1 — fabric-range-reads

**Create:**
- `internal/fabricengine#FileDiff`
- `internal/fabricengine#Fabric.CommittedFileDiffs`
- `internal/fabricengine#Fabric.CommitMessages`
- `internal/fabricengine_test#TestCommittedFileDiffs_ListsFilesThatExistAtTheBase`
- `internal/fabricengine_test#TestCommitMessages_ListsWholeMessagesNewestFirst`

**Uses:**
- `internal/fabricengine#Fabric.CommitLog`
- `internal/fabricengine#Fabric.ReadCommittedDiff`
- `internal/fabricengine_test#TestCommitLog_ListsTheRangeNewestFirst`
- `//internal/fabricengine/doc.go`

**Intent:** Give D3 its two git reads, as general fabric seams any module can take (`PATTERN-fabric-boundary`), so `internal/driftengine` never runs git.

- `FileDiff` holds `Path`, repository-relative and slash-separated, and `Diff`, the file's own diff text.
- `CommittedFileDiffs(base)` lists every file that exists at `base` and that the commits from `base` to the code checkout's `HEAD` change: modified or deleted, read with `git diff --name-only -z --no-renames --diff-filter=MD base HEAD`, so a rename lists its old path and an added file is not listed.
  Each entry carries that file's committed diff, `git diff --no-color --no-ext-diff --no-textconv --no-renames -U3 base HEAD -- <path>`, the same flags `ReadCommittedDiff` uses.
  The list is sorted by path; an empty range gives an empty list, and a `base` that does not resolve is an error naming it.
  Uncommitted edits and untracked files are never listed, as for `ReadCommittedDiff`.
- `CommitMessages(from, to)` lists the whole message of every commit reachable from `to` and not from `from`, newest first, read through the same `CommitDetail` walk `CommitLog` makes; an unresolvable ref is an error naming it.

Both tests are tier1, in the external `fabricengine_test` package beside `TestCommitLog_ListsTheRangeNewestFirst`, built on the same hermetic repository helpers.
`TestCommittedFileDiffs_ListsFilesThatExistAtTheBase` commits, over a base, a modified file, a deleted file, a renamed file and an added file, and asserts the exact listed paths (modified, deleted, the rename's old path) and that each diff holds its own change only.
A second row passes a base that does not resolve and asserts the error names it.
`TestCommitMessages_ListsWholeMessagesNewestFirst` asserts two commits' full messages, a trailer paragraph included, newest first, and a row with an unresolvable `from` asserts the error names it.

**Commit:** `1: fabric-range-reads`
