# Pin adopt handover gaps: beam under root, owner archive, teardown

Tests only; no production code changes.
They close findings `adopt-owner-handover` and `adopt-here-archives` left open under "To check by hand".

## What changes

- A beam with no owner recorded under an unfinished `root` adopt is now pinned as covered, which settles the finding the earlier review marked blocking.
  The beam's owner falls back to the prime's, here another machine, and `adoptrecord.Watch.Covers` reads the beam covered once a running `root` adopt is recorded.
  Two tests show it: one for the watcher's accessor in `internal/orchcli`, one for the dobby's `UnderAdopt` seam in `internal/beamcli`.
  Both read the beam uncovered before the adopt record exists, so the watcher and dobby of such a beam stay up and idle during `lym adopt root`.
  The accessor already read covered, so no fix was needed.
- `TestReadOwnerArchives` gains rows for the records tag against the records branch.
  Each side's tag is listed even when its own origin holds the archived tip under the other side's branch name.
  A tag is judged against its own side's branch, not the decoy.
- A new `Remove` test pins that a teardown beside an owner archive archives the weft tip under its own `archive/` tag.
  The owner archive tag stays as it was on origin, and the worktree is still removed.

## Design decisions

- The two beam-under-root tests build the same fixture: a hub with a beam pair, and a board whose prime owner is another machine.
  Each goes through the real accessor rather than a stub, so the owner fallback is exercised.

## To check by hand

- The decision rows for the owner archive's tips and the adopt hold's skipped steps have no new tests in this change.
  Check that the existing tests cover them.
- The owner archive over a beam's pairs end to end has no new test beyond the teardown one above.
  Check that existing coverage is enough.

All new tests are tier 1 integration tests and were not run by hand.
