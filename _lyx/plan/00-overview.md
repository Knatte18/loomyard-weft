---
format: 5
approved: false
language: go
---

# Plan: batten-hub-ready

Make `lyx batten run <slug>` the hub orchestrator's only driving command.
Today an `awaiting` child fails the batten run, time awaiting PR approval exhausts `Run-Shed`'s watch budget, and teardown destroys the run's friction notes and drive reports.
After this plan batten watches an awaiting child without spending budget, resumes it itself once per `lyx loom approve`, waits for the child's driver to finish its stop report, and tears the pair down with the records committed and archived under a tag on the weft origin.

## Card Index

1 — shedengine-budget-exempt — a Stuck verdict can be marked budget-exempt and is then not counted
2 — shedrun-drive-reports-dir — shedrun declares the durable drive-reports directory
3 — loomengine-durable-friction — friction notes and archives move under _lyx/loom, the lock stays in .lyx
4 — loomengine-driver-strand-name — the driver strand name moves to loomengine so batten can look it up
5 — loom-status-commit-records — the per-transition status commit stages the loom durable dir and drive reports
6 — loom-driver-report-path — drive reports are written into the durable drive-reports directory
7 — loom-commit-records-verb — new lyx loom commit-records verb commits and pushes the run records
8 — loom-driver-teardown-command — the driver's end-of-session command runs commit-records before strand removal
9 — batten-innerrun-seams — approval, driver-liveness and clock seams on InnerRunDeps, wired from prime
10 — batten-innerrun-awaiting-donewait — Run-Shed watches awaiting, resumes per approval, waits for driver exit
11 — fabric-archive-helper — tag and push the weft tip under archive/ before a branch is deleted
12 — fabric-remove-archives — Remove archives the weft tip ahead of its first mutation
13 — fabric-pairbranch-cleanup-archives — RemovePairBranch and Cleanup archive before deleting a weft branch
14 — fabric-sibling-dirty-sentinel — a sentinel for Remove's dirty-sibling refusal
15 — batten-teardown-records-remedy — batten's teardown names commit-records and a resume, never --force
16 — batten-records-integration — end-to-end: awaiting, approval, resume, done, teardown, records archived

## Shared Decisions

### Decision: stage-the-loom-durable-dir-whole

- **Decision:** the status commit and `lyx loom commit-records` stage `_lyx/loom` as one pathspec entry when it holds a file, rather than the friction directory and each archive sibling separately.
- **Rationale:** `StageAndCommit` runs `git add -- <pathspec>`; after reflection renames `friction` to `friction-<ts>`, only a parent entry also stages the old path's removal, and an unstaged removal would leave the pair dirty and make teardown refuse it.
- **Applies to:** cards 3, 5, 7

### Decision: archive-tag-shape

- **Decision:** the archive tag is `archive/<slug>/<first 12 hex of the weft tip SHA>`, created and pushed by one unexported helper, `archiveWeftTip`, in `internal/fabricengine/archive.go`.
- **Rationale:** deterministic from slug and tip, so a resumed teardown on the same tip finds its own tag, and distinct per tip, so a teardown retried after `lyx loom commit-records` archives the new tip without a clash.
- **Applies to:** cards 11, 12, 13, 16

### Decision: every-weft-branch-deletion-archives

- **Decision:** `Remove`, `RemovePairBranch` and `Cleanup` archive before deleting an existing pair's weft branch; `Add`'s rollback does not; `Prune` deletes no weft branch and needs no step.
- **Rationale:** the decision record's scope is every path that deletes a pair's weft branch; reading the code shows `Prune` leaves orphaned weft branches to `Cleanup`, so `Cleanup` is the path the record's "prune" names in practice.
- **Applies to:** cards 11, 12, 13

### Decision: vocabulary-outside-fabric

- **Decision:** new identifiers, strings and comments in `battenshed`, `battencli` and `loomcli` never name either fabric side; the dirty-sibling sentinel is `ErrPairSiblingDirty`, and the archive result fields are `ArchiveTag`/`ArchiveSkippedReason`.
- **Rationale:** none of those packages is in the Fabric Vocabulary Invariant's owner set.
- **Applies to:** cards 7, 9, 10, 12, 13, 14, 15

### Decision: liveness-error-is-live

- **Decision:** on a `done` child, a `DriverAlive` error is logged and treated as a live strand.
- **Rationale:** the wait is already bounded by `driver_exit_grace_s`, so a flaky liveness read costs at most the grace window, where a hard error would fail a finished batten run at its last row.
- **Applies to:** card 10

## verify:

go build ./...
go test ./internal/shedengine/... ./internal/shedrun/... ./internal/loomengine/... ./internal/loomcli/... ./internal/webstercli/... ./internal/battenshed/... ./internal/battencli/... ./internal/shedrecipe/... ./internal/shedbuild/... ./internal/battenrecipe/... ./internal/fabricengine/... ./cmd/lyx/...
go test -tags integration ./internal/shedengine/... ./internal/shedrun/... ./internal/loomengine/... ./internal/loomcli/... ./internal/webstercli/... ./internal/battenshed/... ./internal/battencli/... ./internal/shedrecipe/... ./internal/shedbuild/... ./internal/battenrecipe/... ./internal/fabricengine/... ./cmd/lyx/...
go vet -tags smoke ./internal/loomcli/... ./internal/battencli/... ./cmd/lyx/...
