---
format: 5
approved: false
language: go
---

# Plan: loom start refuses a worktree with an unfinished merge

`lyx loom start` spawned a driver onto a pair left mid-merge by a conflicted hub merge-in, because nothing on the start path checks merge state and the recipe's Preflight row never re-runs on a resume.
This plan adds a read-only, vocabulary-neutral mid-merge probe to `internal/fabricengine` and makes `start` consult it exactly when the invocation would spawn a driver (either arm) or resume a parked ly-drive driver.
A mid-merge pair is refused on the envelope with the new non-retryable kind `merge_in_progress`, listing the conflicted paths and naming the remedy;
a live, working driver is left alone, and `step`, `run`, `shed step` and the Preflight row are unchanged.

## Card Index

1 — merge-in-progress-kind — declare the merge_in_progress refusal kind beside StartNotParkedKind
2 — midmerge-probe — add fabricengine's MidMerge probe with a row per state
3 — start-refuses-mid-merge — refuse a spawn or resume over a mid-merge pair, with help text and overview docs

## Shared Decisions

### Decision: probe-answers-never-default-to-clean

- **Decision:** every probe failure — a pair `Open` cannot open, a record location that cannot be resolved, a conflict-path geometry error, an unmappable conflicted path — is returned as an error, and `start` refuses on it.
- **Rationale:** an undetermined answer is not evidence the tree is clean, and spawning on it reopens the bug.
- **Applies to:** cards 2 and 3

### Decision: gate-keys-on-the-existing-spawn-resume-decision

- **Decision:** `start` probes only when `mustSpawn` is true or a live driver strand has its park marker present, after the run-lock and strand probe and before marker removal, the resume line, or any spawn.
- **Rationale:** Landing's conflict-resolution session runs mid-merge under a live driver, so re-attaching to a working driver must not be refused, and keying on the existing decision adds no second read of the recorded seed driver.
- **Applies to:** card 3

## verify:

go build ./...
go test ./...
go test -tags integration ./internal/fabricengine/... ./internal/loomcli/... ./internal/shedrun/...
go vet -tags smoke ./internal/loomcli/... ./internal/fabricengine/...
