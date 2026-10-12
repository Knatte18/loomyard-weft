## Fixer-report, round 1

### Changes

- F1: in `internal/fabricengine/mergevocab_test.go`, the doc comment of `mergeVocabContainsBareToken` now names `internal/fabricvocab`'s `BareToken` instead of the deleted `bareVocabularyToken`.
  Fixed as proposed; commit `9820aeffd`.
- Gate finding: the same comment was fixed-column wrapped.
  It is now broken at sentence boundaries, one sentence per line, in a follow-up commit.
  Comment-only changes, so no test was added; `gofmt -l` reports nothing.

### Disputed

None.

### Deferred

None.
