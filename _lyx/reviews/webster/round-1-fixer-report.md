# Round 1 fixer report

Every finding in `round-1-review.md` is fixed, one commit each, none pushed.

## Fixed

- F1 (BLOCKING, consistency) — re-broke the new and changed Go comments at sentence and clause boundaries in `configsync.go`, `bolt.go`, `configreg.go`, `clone.go`, `fabric.go`, `configcli.go` and `boardcli/cli.go`.
  Untouched legacy wrapped lines were left alone, and the `fabric.go` comment no longer quotes the old `not initialized here` message.
- F2 (BLOCKING, consistency) — `TestDispatchHubWideBoard` and its rows, and `TestBolt_CommitWritten`, now call `t.Parallel()`.
  The `CommitWritten` subtests stay serial because they share one repo.
- F3 (MEDIUM, scope) — `TestBolt_CommitWritten` gained subtests for a deleted path, an empty path list and `SkipGit`.
  This was a coverage gap, so tests were added; the deletion subtest passes against real git.
- F4 (LOW, consistency) — `docs/shared-libs/configengine.md` names `boardengine.LoadConfig`'s `errors.Is` check and its new message, and lists the substring-matching callers without a count.
- F5 (LOW, consistency) — the `lyx fabric reconcile` `Long` text describes the hub-wide config repair, the `_board` commit and the `hub_config`, `hub_config_detail` and `warnings` envelope keys.

## Verification

`go build ./...`, `go vet ./...`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...` pass after the last commit.

## Deferred

Nothing deferred.
