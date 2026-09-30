# Card 5 — loom-status-commit-records

**Edit:**
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#reviewsHoldFile`
- `internal/loomcli#commitStatusDeps`
- `internal/loomcli#loomCommitStatusDeps`
- `internal/loomcli/wiring_commitstatus_test.go`
- `internal/loomcli/wiring_commitstatus_integration_test.go`

**Uses:**
- `internal/loomengine#LoomDurableDir`
- `internal/loomengine#LoomDurableDirRel`
- `internal/loomengine#LoomFrictionDir`
- `internal/shedrun#DriveReportsDir`
- `internal/shedrun#DriveReportsRel`

**Intent:**
Loom's per-transition status commit also stages the run records, so friction notes written during a producer are committed by the transition that follows it, through the same route `_lyx/reviews/` already takes.

`statusCommitPathspec` appends, each under the same "holds at least one non-directory entry anywhere beneath it" rule it applies to the reviews directory:
- `loomengine.LoomDurableDirRel()` when `loomengine.LoomDurableDir(location)` holds a file.
  The whole `_lyx/loom` directory is staged rather than the friction directory alone, because `StageAndCommit` runs `git add -- <pathspec>`: staging the parent carries the friction directory, every timestamped archive sibling, and an archive rename's removal of the old path in one entry, where a friction-only entry would miss the removal once the directory is renamed away and leave the pair dirty for teardown.
- `shedrun.DriveReportsRel(location, runID)` when `shedrun.DriveReportsDir(location, runID)` holds a file.

Rename `reviewsHoldFile` to `holdsFile`, since it now answers for three directories, and update its callers and doc comment.
Update `statusCommitPathspec`'s, `commitStatusDeps.Commit`'s and `loomCommitStatusDeps`'s doc comments, which describe the pathspec as the status file plus the review round record.

Tests in `wiring_commitstatus_test.go`, extending the existing `statusCommitPathspec` table over a temp-dir location: the loom durable directory and the drive-reports directory each appear exactly when they hold a file (at any depth, including only an archive sibling under `_lyx/loom/`), and are omitted when absent, empty, or holding only empty subdirectories.

Tests in `wiring_commitstatus_integration_test.go`, over its existing `realSeamFixture` (a real hubforge pair and the real `loomCommitStatusDeps`), beside the reviews-directory cases they mirror:
- a friction note under `loomengine.LoomFrictionDir` and a drive report under `shedrun.DriveReportsDir`, written before a transition, land in the weft HEAD commit and leave `_lyx/loom` and the drive-reports directory clean;
- a committed friction directory renamed to a timestamped `friction-<ts>` sibling (the reflection archive) is recorded by the next transition as both an addition of the sibling and a deletion of the old path, and leaves `_lyx/loom` clean.
These are the proof that the records pathspec carries both kinds of record onto the weft tip, which card 16's end-to-end test stubs out.

**ImpactSummary:** The per-transition commit's pathspec grows by two optional entries; the commit, push and mid-merge dispositions are unchanged.
