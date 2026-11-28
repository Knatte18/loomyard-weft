# Fail every task's tier0 run on a Go file gofmt would rewrite

A mis-formatted Go file could pass a task's own test runs.
The formatting guard lived in `cmd/lym` and was marked `//lym:only parent-align finalize`,
so among `--gate` calls it ran only under those two landing keys.
It now lives in `internal/testkit/gofmt_test.go`, keeps `//lym:guard` and drops `//lym:only`,
so every task's tier0 gate runs it, whatever the task changed.

The guard is also stricter and its failure says how to fix the file:

- A file that differs from its gofmt form only at its end, by a missing final newline or extra trailing blank lines,
  now fails. Before, it passed.
- Each formatting finding names `lym loom fmt <file>` as the command that rewrites the file.

The help text of `lym loom fmt` and the `internal/testkit` package doc point at the guard's new place.

## Design decisions

- The check reads the whole tree, not only changed files, so it needs no diff scope.
  It covers every Go file of the module whatever its build tags, except under directories the shared scan skips,
  which include `testdata` fixtures.
- It lives in `internal/testkit` rather than `cmd/lym` so that the test binary each task has to build for it stays small.
- It compares each file with `go/format.Source`'s output, so it needs no `gofmt` binary.

## To check by hand

- A file that differed from its gofmt form only by a missing final newline or extra trailing blank lines
  passed the guard before this change and now fails.
  Check the finding `gofmtFinding` (`internal/testkit/gofmt_test.go`) returns when no compared line differs,
  and the table rows for those two cases.
- No run was made with a deliberately mis-formatted file in a change that leaves `internal/testkit` untouched,
  to see a task's tier0 gate fail on it.

## Merge danger

**Door:** two-way
**Blast radius:** gate
