## Changes

- **F1** fixed as proposed.
  `moveWeftContent` returns `(committed bool, err error)` instead of a sha, and reads the commit through the `weftHeadSHA` seam.
  A failed read after the move commit returns `committed` true with the error, so `moveWeft` keeps the record and names the re-run instead of rolling back.
  A `TestAnchorChange` row fails the read through `SetWeftHeadSHAForTest` and asserts landed true, the re-run named, the record standing and the commit kept.
- **F2** fixed as proposed.
  `MoveInProgress` names a held lock as in progress only when `recordNamesLiveHolder` holds, a check `heldMoveRefusal` now shares; otherwise it words the stop as starting.
  Rows: `LockHeldByLiveRecordProcess` and `LockHeldOverStaleRecord` in `TestMoveInProgress`.
  The `LockHeld` rows of `moverecord_test.go` and `addparent_integration_test.go` asserted the old wrong wording and were corrected.
  `doc.go` states the rule.
- **F3** fixed as proposed.
  `readRecordedAnchor` returns an error naming the own marker when its read fails for any reason but not-exist.
  Test: `TestReadRecordedAnchor_UnreadableOwnMarkerIsErrorNamingFile`, an own marker that is a directory.

Commits: F3, then F1 and F2 together (both change `anchorchange.go` and the second does not build without the first), then a follow-up for the self-check's F2 finding.
Gate: `lyx gate test` tier 0 and tier 1 of `./internal/fabricengine`, `./internal/lyxcwd`, and tier 1 of `./internal/hubgeom`, `./internal/fabriccli` are green.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW: `doc.go` described a held lock as always a change in progress. Fixed as proposed; `doc.go` now states the live-holder rule.
- fix-proposal, LOW: the starting stop named the stale record's B as the re-run target, which can differ from the holder's target. Fixed as proposed; the stop from `MoveInProgress` names no target and says to run the verb again, and `heldMoveRefusal` keeps naming its own target.
- fix-proposal, LOW: `ENOTDIR` is not `fs.ErrNotExist`, so an own-marker path under a regular file errors instead of falling back. Disputed, see below.

## Disputed

- fix-proposal LOW (ENOTDIR), ground `every fix worse`: the marker's parent is the records sibling directory derived from the hub, which the resolver has already established; a regular file there is a state the code's own guarantees rule out.
  Handling it needs a platform error check for that impossible state, which `code-quality` says not to add, and it would turn a loud error into silence for the same unreadable-marker class F3 closes.

## Deferred

None.
