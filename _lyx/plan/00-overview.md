---
format: 5
approved: false
language: go
---

# Plan: Creel gets its own sandbox suite

Creel's sandbox scenarios, S9 and C1 to C13, move out of `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md` into a self-contained `tools/sandbox/SANDBOX-CREEL-SUITE.md` with their ids and wording kept, and the shuttle suite keeps S1 to S8.
The sandbox tool gains a `creel-suite` subcommand running that file through the shared `runSuite`, with POSIX and Windows launchers, and `docs/sandbox-howto.md` names the suite and its launchers.
A board note gives the rewrite of C13's stacked half an owner.

## Card Index

1 — creel-suite-file — write SANDBOX-CREEL-SUITE.md with S9 and C1-C13, cut the shuttle suite to S1-S8, file the C13 board note
2 — creel-suite-subcommand — embed the creel suite as `creelSuite` and dispatch it from a `creel-suite` subcommand, with table-test rows
3 — creel-launchers-and-docs — add the creel-suite launchers and name the creel suite in docs/sandbox-howto.md

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
