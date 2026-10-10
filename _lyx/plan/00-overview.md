---
format: 5
approved: false
first_card: 12
language: go
---

# Plan: reed hygiene — a reachable Debug trace, a refused over-long socket path, hermetic servers and a backed-off poll mode

Three reed fixes land as one task, closing #495.
The retired generation's eleven cards are in the code: the watchdog daemon's Debug lines reach a log the operator can switch on, its per-session watchers drive the told tmux binary so one shim counts every spawn, the resize watch loop's poll mode backs off under `PATTERN-daemon-wakeups`, an over-long socket path is refused at boot with `TMUX_TMPDIR` named, reed's own servers stop reading `~/.tmux.conf` and strand panes stop starting login shells, the resize signal entry is installed at boot so a strand-less session's watcher leaves poll mode, and the idle-CPU benchmark is re-run with the children identified.
The pull request's review approved that work and rejected the branch on one finding, F1: the branch and `main` each added one top-level test to `cmd/lyx` and each raised the package's row in `cmd/lyx/testdata/test-budget.yaml` by one, with textually identical edits, so the merge is clean but the merged package holds one test more than the row allows and `TestTestBudget_PackagesStayWithinBudget` fails after Finalize.
This generation is the one card that fixes it, and changes nothing else.
The one carry-over review finding, the socket path measured before symlink resolution, is what `resolvedSocketDir` in `internal/reedengine/socketfile.go` does, and it is not scope.

## Card Index

12 — cmd-lyx-budget-row-after-merge — the `cmd/lyx` test budget row admits the merged package's top-level test count

## Shared Decisions

The retired generation's decisions below still describe the code and bind nothing this generation changes; `test-budget-rows` gains the row card 12 raises.

### Decision: goos-as-parameter

A new pure function whose answer depends on the platform (`unixSocketPathLimit`, `serverSpawnArgv`) takes the GOOS as a parameter and its production caller passes `runtime.GOOS`, so one untagged table pins both branches; a method that only gates on the platform keeps reading `runtime.GOOS` directly, as `pinGeometryOptionsLocked` does today.

### Decision: boot-refusals-in-the-pre-tmux-block

The socket-path and shell refusals sit in `ensureServerAndSessionLocked` right after `validateBootConfig` and before `probeCapabilityLocked`, so `TestUp_BootValidation`'s zero-tmux-calls assertion covers them as rows; `validateBootConfig` keeps its signature, since `zoom.go` also calls it for a pure config check that must not resolve paths.

### Decision: boot-tail-extracted-once

The post-spawn pin block lives in `pinBootOptionsLocked`, so the boot path's re-sign happened once and the hook install is reachable by a fake tmux.

### Decision: probe-server-bound

The capability probe's `list-commands` may start a transient server on reed's socket ahead of `new-session`, and without `-f` that server reads the operator's config.
A config that creates a session keeps it alive, `sessionlessSocketHolderPersists` never reaps a server that lists a session, and reed's `new-session` would join it, against D7's rule that reed's own servers read no `~/.tmux.conf`.
So the probe's two invocations carry `-f /dev/null` as well, through a config-file field on the probe's copy of the engine's `TmuxCmd`, and every other engine call keeps its argv.

### Decision: test-budget-rows

The retired generation raised `internal/reedengine`'s row to 247 and filled `internal/reedcli`'s spare test; the review-fix commit for the root hook's log level added one `cmd/lyx` test and raised that row to 74.
`main` added one `cmd/lyx` test under the same row value, so card 12 sets the row to 75, the merged count.
The budget is a ceiling, a count below its row passes, so the row holds on the branch before Finalize's catch-up merge with `main` and on the merged tree after it.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
