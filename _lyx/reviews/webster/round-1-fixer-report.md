# Round 1 fixer report

## Fixed

- F1 (LOW, consistency): re-broke the hard-wrapped comments at sentence and clause boundaries, in commit `b261936da`.
  Files: `internal/pattern/pattern.go`, `internal/pattern/doc.go`, `internal/loomengine/prompt.go`, `internal/burlerengine/geometry.go`, `internal/hubgeom/hubgeom.go`, `internal/lyxcwd/enforcement_test.go`, `internal/lyxcwd/vocabscan_test.go`.
  `go build ./...` and the tests of the five touched packages pass.

## Deferred

Nothing deferred.

## Noted, not changed

`gofmt -l` lists `internal/pattern/doc.go` for the indented entry-line example, which gofmt would reformat as a list item.
That line was not part of F1 and was left as written.
