---
format: 5
approved: false
language: go
---

# Plan: cleanup-1012

Three independent cleanups from recent PR-Gate reviews, one card each, per the decision record's D1–D3: an opt-in `-no-cutoff` flag on `tools/tokencount` that drops the archive-tag lower bounds; reader segments in `IsReadOnlyCommand` that reject an unquoted parenthesis and a dollar expansion, with the reader rationale documented in `internal/fabricengine/doc.go`; and the vocabulary scan's test-local records-name strip removed in favour of `fabricvocab.BareTokenInRootDoc` alone.

## Card Index

1 — tokencount-no-cutoff — add `-no-cutoff` to tokencount, zeroing the run bound and the shed span's older-tag clamp, with its header line and doc comment
2 — reader-paren-dollar — reject an unquoted parenthesis and a dollar expansion in a reader segment, and document the reader rationale in `doc.go`
3 — vocab-root-replaceall — remove `recordsRepositoryName` and its `ReplaceAll` from `fabricVocabularyFailures`, with two README fixture rows

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
