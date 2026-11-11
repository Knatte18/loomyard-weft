# Pin the adopt handover's open checks in tests

The landings of `adopt-owner-handover` and `adopt-here-archives` left a set of points under "To check by hand".
This change adds tests for those that no test pinned.
It changes no production code: the BLOCKING point reads covered, so it needs no fix.

## What changes

- **A beam with no owner of its own, under an unfinished `root` adopt.**
  Two integration tests run over a hubforge hub whose board records another machine as the prime's owner,
  and records beam project `b` with no owner.
  - `TestUnderAdoptWatch_CoversABeamWithNoOwnerOfItsOwnWhileARootAdoptIsUnfinished` in `internal/orchcli` pins the orch watcher's side.
    `beamOwnerElsewhere` reads `b` owned elsewhere, and the accessor `underAdoptWatch` builds asks about slug `b`.
    That accessor reads `b` uncovered with no record, and covered under a running `root` adopt.
  - `TestHubDobbyDeps_ABeamWithNoOwnerOfItsOwnIsUnderARootAdopt` in `internal/beamcli` pins the dobby's side the same way, through the seams `hubDobbyDeps` fills.
    Its `Owner` reads the prime's foreign owner, and its `UnderAdopt` turns true under the adopt.

  The engine tests already pin that a watcher or dobby whose `UnderAdopt` reads true stays up and idle.
  These tests close the chain from a real board and record to that seam.
- **Owner archive reads against a decoy branch.**
  Two rows in `TestReadOwnerArchives` push the archived tip to the side's origin under the other side's branch name for the slug.
  A code tag stays listed although a branch named for the records branch on the code origin holds it.
  A records tag stays listed although a branch named for the code branch on the records origin holds it.
  So each tag is judged against its own side's branch alone.
- **Teardown beside an owner archive.**
  `TestRemove_LeavesAnOwnerArchiveTagOutOfTheTeardownArchive` owner-archives a pair's unpushed weft tip and then runs `Remove`.
  The teardown tags the tip `archive/<slug>/<sha12>` rather than reusing the owner archive tag.
  The records origin holds exactly that one `archive/` tag, the owner archive tag stays at the tip, and the worktree is gone.

## Design decisions

- The remaining findings already had tests that landed with the code they name, so this change adds none for them:
  - The hold's skipped pause, archive and pass starts are pinned by `TestRun_ForeignOwner`'s `AnAdoptInFlightKeepsTheDobbyUpAndIdle` row and by `TestRun_AnAdoptHoldSchedulesNothingUntilItLifts`.
  - The owner archive's absent, equal, ahead and diverged tip rows are pinned by `TestArchiveOwnerTips`.
  - The pairs the archive covers are pinned end to end by `TestArchiveOwnerTips_CoversTheWorktreesOwnPairAndTheRunsUnderIt` in `internal/beamcli`.
  - The exclusion of a teardown `archive/` tag from the `--archives` list is pinned by `TestReadOwnerArchives`'s `ATeardownArchiveTagIsNeitherListedNorFetched` row.
- The two folded notes are read as the cases those existing rows left open:
  the read against a branch named for the other side, and an owner archive tag present during a teardown.
- The beam tests write `board.json` directly, since the beam verbs that set the beam flag also record an owner.
- The decoy push uses `--force`, because a branch of that name may already sit on the origin.
