## Changes

- F1 (MEDIUM), fixed as proposed, commit `fabricengine: fix F1, F2`:
  - `TestBolt_CommitWritten` gained a row where the write returns only a path neither on disk nor in HEAD, asserting no commit, no error and HEAD unchanged.
  - It gained a second row on a repo with no commits, covering the `head == ""` branch: the written file is committed and the missing one left out.
  - `TestBolt_PullThenCommitWritten` gained the same nothing-stageable row, exercising the `len(stageable) == 0` continue.
  - No new test function was added.
- F2 (LOW), fixed as proposed, same commit: the deletion subtest in `TestBolt_CommitWritten` asserts HEAD's name-status equals `D\tseed.md`.
- F3 (LOW), fixed as proposed, commit `fabricengine: fix F3`: the `stageablePaths` doc comment states that paths are file paths and a directory HEAD holds is not recognised as held.
- Self-check follow-up, commit `fabricengine: fix self-check`: the `TestBolt_CommitWritten` doc comment names the new subtests.

`lyx gate test --tier 1 ./internal/fabricengine` passed, with the new subtests in the log.

## Gate repair

The verify run failed `TestCLI/ListAndFindText` in `internal/boardcli`: `list --sort revised` returned `[later soon]`, want `[soon later]`.
The test depends on whole-second `revised` stamps of two upserts landing in the same second, so it fails whenever a second boundary falls between them; the package is untouched by this round.
Commit `boardcli: fix gate failure`: the test now upserts both entries with the same explicit `revised` stamp before asserting the order.
It passed three repeated runs at tier 1 afterwards.

## Self-check

- fix-consistency, LOW: the `TestBolt_CommitWritten` doc comment lagged the new subtests; fixed as proposed.
- fix-proposal, LOW: the unborn-HEAD row may not fail without the `head == ""` branch if `FileAtRevision` on an empty revision returned `ErrPathNotAtRevision`; no change, the row still covers the unborn path end to end and the premise is not shown to be a defect.
- fix-proposal, LOW: the F2 assertion depends on subtest order within one shared repo; no change, it is the existing fixture style.
- fix-proposal, LOW: the F3 comment is accurate and breaks nothing; no change needed.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
