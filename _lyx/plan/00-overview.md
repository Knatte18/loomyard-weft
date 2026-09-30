---
format: 5
approved: true
language: go
---

# Plan: Webster survives a parent merge between a fork's commit and record-batch

A parent merge-in that lands after a fork commits but before `record-batch` (or `recover-batch`) consumes its report wedges the run: the report's `head_sha` no longer equals HEAD, so record-batch refuses, and begin-batch/recover-batch refuse because an OK report exists.
This plan makes the head cross-check accept HEAD when only merge commits sit between it and the report's `head_sha` on the first-parent chain, records the batch at the report's own commit, refuses while a git merge is in progress, and has `lyx fabric merge-in` warn (never refuse) while a Webster run is in flight.

## Card Index

1 — head-reconcile-helpers — add the merge-only head walk and the mid-merge probe beside `headSHA`
2 — record-batch-tolerates-merge — record-batch uses both helpers and records the batch at the report's head; package doc describes the rule
3 — recover-batch-tolerates-merge — recover-batch follows the same rule; package doc covers both verbs
4 — parent-merge-wedge-scenario — end-to-end begin → fork commit → parent merge → record → next begin test
5 — run-in-flight-predicate — export `RunInFlight` over webster's own `state.json` and `outcome.yaml`
6 — merge-in-warns-mid-run — `lyx fabric merge-in` warns mid-run; `Long` help and the ly-drive rule document it

## Shared Decisions

- Every new git access is a read-only query through `internal/gitrepo` (`MergeHeadPresent`, `CommitParents`, `CurrentSHA`), which the Fabric Git Invariant exempts; nothing adds a git mutation.
- The refusal text keeps the existing `head_sha %q does not match the worktree's actual HEAD %q` phrase so tests pinning it pass unchanged, and appends the merge-only rule.
- New git-backed tests are `//go:build integration` in packages whose `TestMain` already calls `gitkit.HermeticGitEnv()`; the fabriccli fixture is `hubforge.NewHub`.
- `CONSTRAINTS.md` and `manifest/roadmap.md` are unchanged: no new cross-cutting invariant, and this is a bugfix.

## verify:

```bash
go build ./...
go test ./internal/websterengine/... ./internal/fabriccli/... ./cmd/lyx/...
go test -tags integration ./internal/websterengine/... ./internal/fabriccli/... ./cmd/lyx/...
```
