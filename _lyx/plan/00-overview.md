---
format: 5
approved: false
language: go
first_card: 11
---

# Plan: shuttle-send — sends that land, and stops that are recorded

A text shuttle sends into an agent pane must land, and when it cannot, the run retries instead of halting (#442, #443, and the "no send into a running turn after a restart" case).
A shuttle run lyx itself stopped is recorded `stopped` through one shuttle stop verb, so a later attach probe never halts on it (#444).
Cards 1 to 10 built that and are in the code.
The PR gate rejected PR #474 on one finding: `PATTERN-shuttle-stop` in `PATTERN.md` names only the loom driver's exception, while `orchcli` also removes the orch strand, which has a shuttle run record, directly.
This generation fixes only that entry's wording; no Go code changes.

## Card Index

11 — shuttle-stop-orch-exception — `PATTERN-shuttle-stop` names `orchcli`'s removal of the orch strand as its second exception

## Shared Decisions

### Decision: verify-scope

The operator's finding limits verification to the PATTERN scans' own tests, not the full suite.
No card touches Go code, so `## verify:` builds the module and runs the tiers over `internal/pattern` alone, the package whose tests read and check `PATTERN.md`.

## verify:

```
go build ./...
go vet -tags integration ./internal/pattern/...
go vet -tags tmux ./internal/pattern/...
go vet -tags llm ./internal/pattern/...
go test ./internal/pattern/...
go test -tags integration ./internal/pattern/...
```
