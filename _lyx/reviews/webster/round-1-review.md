---
verdict: APPROVED
---

No findings.

Range: `product.parent` is `main` and no `_lyx/loom/rework/` round exists, so the range is `git diff $(git merge-base main HEAD)..HEAD`, the three card commits.

Focus questions settled, each against code I read:

- `cloneTasks` clones `Labels`, `Issues`, `DependsOn` and the `Status` pointer.
  The snapshot is taken right after `Load`; `checkRunLocks` runs after `fn` and before `Save`.
  A `noWrite` result returns before the check, and a refused write returns before `Save`; every `TestRunLock` refused row asserts `board.json` is byte-identical.
- `scopeChanged` compares `Title`, `Kind`, `Recipe`, `Isolated`, `Brief`, `Body`, `ShortName`, `Labels`, `Issues` and `DependsOn`; `slices.Equal` treats nil and empty alike.
  An absent entry is refused.
  The prune exception strips only slugs that were `done` in `loaded` and are absent from `after`; the prune row and the `depends_on` refusal rows pin both sides.
- `RunLockedError.Error` names slug, status, the note way forward, `lyx batten status <slug>` from the prime worktree and `lyx board set-status`; it names no denied command.
  Its claim that `batten status` refuses on a slug with no seed matches the verb's help text.
- The claim is the first statement of the `CreateWorktree` closure in `internal/battencli/wire.go`, composed through `boardengine.RunStatus`, and a failed open or write warns and continues.
  Nothing lands in `battenshed` or `shedengine`.
- `openHubBoard` has no remaining reference.
  `OpenHub` reads from `fabricengine.BoardDir(hubPath)` only, and `SetRunStatus` skips absent and done entries inside the critical section.
- Test economy: `TestRunLock`, `TestSetRunStatus` and `TestWire_CreateWorktreeClaimsTheBoardEntry` each cover behavior no other test covers.
  The intake row and the `lifecycle_integration_test.go` step extend existing tests.
  The `test-budget.yaml` bumps (`internal/boardengine` +2, `internal/battencli` +1) match the new top-level tests.
- Docs match the code: `internal/boardengine/doc.go` `# Run lock`, `internal/boardcli/doc.go`, the `internal/battencli/cli.go` package comment and the board `SKILL.md` section.
- Card 2's Delete group (`openHubBoard`) is gone with no stale caller.

Gates run in this worktree: `go build ./...`, `go test ./...`, `go test -tags integration ./...`, `go vet -tags integration ./...` and `go vet -tags tmux ./...` all pass.
