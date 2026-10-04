# Round 1 fixer report

All three findings are fixed, one commit each, with the untagged tests of `boardengine`, `boardcli` and `selfreportengine` green after each.

## Fixed

- **F1** (`83579a2e5`): `internal/selfreportengine/inbox.go` — rewrote the `ListOpenIssues` and `CommentAndClose` doc comments one sentence per line.
- **F2** (`fde97c1b5`): `internal/selfreportengine/selfreport.go` — rewrote the changed package doc, `targetRepo` and `callTimeout` comments, and the comments in the new `repoClient` and `classifyCallError`, with semantic line breaks.
  The comment inside `CreateIssue` that this diff did not touch keeps its old wrapping, as `PATTERN-comment-line-breaks` says untouched lines are not rewrapped.
- **F3** (`7cb8cd2db`): `internal/boardengine/store.go` — `Load` defaults `Labels` to an empty list; `TestLoadNilLabelsNormalization` in `store_test.go` covers it.

## Not changed

- `doc.go`'s Concurrency and Storage sections carry the old wrapped package-doc text moved verbatim from `board.go`; the pattern calls such text legacy and the review recorded no finding on it.
- `internal/boardengine/legacy_test.go` fails `gofmt` on one line that predates this branch.

## Deferred

None.
