---
format: 5
approved: true
language: go
---

# Plan: git-robustness

Five git defects stop or stall runs, and each is fixed at its source.
A cached go-git handle reads a stale pack index after a repack (#468), so every go-git read in `internal/gitrepo` goes through one helper that retries the whole read, after `HeadContains` gains a committer-time bound on its walk.
Every git remote command runs under a deadline and a low-speed limit and classifies a timeout as a transport failure (#458), sharing verifyrun's process-group kill, which moves to `internal/proc`.
The hub's object stores stop auto-gc'ing, and fabricengine runs a foreground gc at pair removal.
`lyx batten run <slug> --window` refuses a done slug before opening a window (#447).

## Card Index

1 — headcontains-bounded-walk — `HeadContains` stops 24 hours of committer time before the target
2 — gitrepo-retrying-read — route every go-git read through one whole-operation retrying helper, the sole `goGit()` caller, with its PATTERN entry
3 — proc-group-kill — move verifyrun's process-group kill into `internal/proc` for gitexec to share
4 — gitexec-remote-bounds — a deadline, a process-group kill and the low-speed environment on every remote git command, timeouts classed as transport failures
5 — hub-store-gc — `gc.auto=0` and `maintenance.auto=false` on both hub stores, and a best-effort foreground gc after a pair's removal, with its PATTERN entry
6 — batten-window-done-refusal — `--window` runs the shared status check and refuses a done slug before opening a window

## Shared Decisions

- No timeout, gc threshold or slack is operator configuration; `remoteDeadline` and `gcAutoThreshold` are variables only for tests, set through `*ForTest` helpers in `export_test.go` files by tests that are not parallel and name the global state.
- Tests that spawn git sit in the `integration` tier under the packages' existing `gitkit.HermeticGitEnv()` TestMains; no test waits out a production deadline.
- `PATTERN.md` sits near `MaxOverviewBytes`: card 2 frees room for both new entries, and every new entry keeps its detail in its background file.
- Each card that adds top-level tests raises its package's row in `cmd/lyx/testdata/test-budget.yaml` in the same commit.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
