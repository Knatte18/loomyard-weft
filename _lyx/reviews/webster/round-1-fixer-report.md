```yaml
round: 1
fixed: [F1, F2, F3, F4, F5]
deferred: []
```

## What changed

Each finding is one commit on `run-pushes`, none pushed.

- **F1** (BLOCKING, consistency): reflowed every new or changed Go comment the review listed so each sentence starts its own line, in `internal/fabricengine` (`pushanchored.go`, `doc.go`, and the three test files) and `internal/fabriccli` (`spawn.go`, `weft_verbs.go`).
  `PushPairAnchored`'s doc comment no longer leans on `PushAnchored` for its retry and lock-wait behavior; it states both itself.
  Untouched legacy lines are left as they were.
- **F2** (LOW, consistency): `statuscommit`'s `Deps.Push` comment, its package doc and the push-warns bullet no longer say the push error names a side or that the push carries the task branch, since the core is shared with batten's records-only wiring.
  `loomcli`'s `commitStatusDeps.Push` comment keeps the claim, limited to what the loom entry does: a failed push names the side, a busy lock names none.
- **F3** (LOW, design): removed the unreachable `!locked` branch from `AcquireWriteLockWithin`; the call now discards the bool, which is always `true` when the error is nil.
- **F4** (LOW, scope): two more steps in `TestPushPairAnchored_PushesBothSidesRetriesAndReportsEachSide`, no new test function:
  a remote with an always-declining `pre-receive` hook returns the bare rejection after the one retry with the bare and local HEAD unmoved, and an unborn code side (an orphan branch) is skipped while the records side still pushes.
  Removing the unborn-HEAD skip from `PushPairAnchored` makes the new step fail, checked by hand and then restored.
- **F5** (LOW, consistency): the `lyx loom` help sentence for `commit-records` now says the task branch is pushed along with the records, naming no side.

## Verification

- `go build ./...`, `go test ./...` and the plan's `-tags integration` package set all pass after the last commit; `go vet -tags llm ./...` is clean.
- `go test -tags tmux ./internal/loomcli/...` was not run: no change in this round touches a tmux-tier file, and the round was not set up with a tmux server.
- `gofmt -l` still lists `internal/fabricengine/pull.go`, `internal/burlerengine/engine.go`, `internal/pattern/doc.go` and `internal/shell/posix.go`; none is in this diff.

## Not fixed

The retry helper's fetch-failure return (a rejected push followed by a failing fetch) still has no test.
The review named it as left out: a local bare cannot reject a push and then fail a fetch without a hack.

## Deferred

None.
