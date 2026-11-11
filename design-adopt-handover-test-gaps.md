# Adopt handover: pin the beam-under-root cover, the hold's skipped steps and the owner archive rows in tests

`adopt-handover-test-gaps` · Task · bug, dobby, fabric, tender · done

## Seen (board tidy, 2026-11-11)

`adopt-owner-handover` landed as `0198b955` with four findings its review left open under "To check by hand":

- A beam with no owner recorded under an adopted `root` may not be covered by the under-adopt accessor in `internal/adoptrecord/watch.go`; the review marked it BLOCKING.
  The accessor's comment says such a beam takes its parent's subject, `root` above the top, so it reads covered;
  no test shows that the watcher and dobby of such a beam stay up and idle during `lym adopt root`.
- The adopt hold in `internal/dobbyengine/handover.go` skips steps (pause, archive, pass starts) that no test pins.
- The decision rows for which tips the owner archive takes (origin absent, equal, ahead, diverged) in `internal/fabricengine/ownerarchive.go` have no test rows.
- The pairs the owner archive covers, in `internal/beamcli/ownerarchive.go` and `internal/fabricengine/ownerarchive.go`, are untested end to end.

## Idea

Settle the BLOCKING point first: a test of `adoptrecord.Watch.Covers` for a beam with no owner of its own under an unfinished `root` adopt,
and a fix if it reads uncovered.
Then add the missing test rows for the hold's skipped steps and the owner archive's decision rows, and one end-to-end test of the archive over a beam's pairs.

## Also pin (folded in)

## From note records-tag-vs-branch-untested

Seen: the landing `adopt-here-archives` lists this under "To check by hand".
No test pins how `lym adopt --here` treats the records tag against the records branch.
Idea: add a test row for each way the two differ.

## From note teardown-archive-untested

Seen: the landing `adopt-here-archives` lists this under "To check by hand".
No test pins the exclusion of an owner archive during teardown.
Idea: add a test that a teardown leaves the archive out and still removes the worktree.

