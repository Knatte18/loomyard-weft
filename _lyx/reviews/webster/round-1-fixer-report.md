# Webster-Burler round 1 — fixer report

Every finding in `round-1-review.md` is fixed, one commit per finding, each green before the next.
After the last commit, `go build ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration ./...` all pass, and `gofmt -l` over every file the range adds or modifies reports nothing.

## Fixed

- **F1 (MEDIUM)** — `639596451`.
  Rejoined every mid-phrase break in the range's new and changed Go comments, so each sentence sits on one line and breaks remain only at sentence ends or independent-clause boundaries.
  The changes cover the cmd/lyx scans, the fabricengine sentinel files, gitkit, the gitoracle package, and the test files the review lists.
  Comments moved verbatim from legacy files were left alone.
  Re-running the review's detector leaves only deliberate clause-boundary breaks (a comma before `so`, `and` or `since` that opens its own clause).
- **F2 (LOW)** — `11478faaa`.
  Restored `bad && !pathAllowlisted(...)` in `cmd/lyx/tierpurity_test.go`.
- **F3 (LOW)** — `1011c0f0d`.
  The `t.Fatalf` messages in `internal/gitrepo/parity_test.go` and `gogit_test.go` now name `gitoracle.CurrentSHA()`, `gitoracle.ChangedFilesSince()` and `gitoracle.CurrentBranch()`.
  A grep for the deleted `oracle*`/`errOracleNoCommits` symbols now returns no hits.
- **F4 (LOW)** — `6e3d29621`.
  `internal/gitkit/doc.go` opens by naming its helpers by group (spawn, query and commit helpers, config seeding, the hermetic environment, `CopyRepo`) and says every export but `CopyRepo` is unpinned, which replaces the stale four-name lists.
- **F5 (LOW)** — `08dcf7774`.
  `internal/hubforge/doc.go`'s export sentence now also names `TestShortname`, `SeedConfig` and `SeedFabricConfig`.

## Deferred

None.
