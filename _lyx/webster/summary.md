# cleanup-1012: tokencount cutoff, reader segment guards, vocab scan strip

Batch 01 (`01-tokencount-no-cutoff`) finished `done` at head `83a2a47668bbcfb082589a7557d36010f6067606`.
It covered the plan's three cards, each committed on its own.

- Card 1 added the opt-in `-no-cutoff` flag to `tools/tokencount`, with its package doc.
- Card 2 made `IsReadOnlyCommand` reject an unquoted parenthesis and a dollar expansion in a reader segment, via `readOnlyReaderSegment` in `internal/fabricengine/refscanner.go`, with the reader rationale in `doc.go` and new `TestIsReadOnlyCommand` rows.
- Card 3 removed `recordsRepositoryName` and its `ReplaceAll` from `fabricVocabularyFailures`, and added two README fixture rows in `vocabscan_test.go`.

## Deviations

`record-batch` reported one file outside the declared file-ops: `internal/fabricengine/refscanner.go`.
It is informational and holds card 2's code change.
