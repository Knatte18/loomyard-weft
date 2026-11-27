---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
---

# Code review round 1: fixer report

## Changes

- F1: `boardSyncRecorder.record` in `internal/hubreconcile/testmain_integration_test.go` counts the sync, releases the lock and returns `boardengine.SpawnSync(boardPath)`.
  Its doc comment says so.
- F2: the broken-config row of `TestEnsureOrWarn_WarnsWithTheWayForward` repairs the file, calls `EnsureOrWarn` again and asserts no way-forward warning and a stamp.
  The row has its old name back, and the summary's "Retry walks removed" bullet no longer lists it.
- F4: the summary's net test-line figure is re-derived from `git diff --stat 2326ef4da..HEAD`: 390 added, 428 removed, 38 fewer.
- F5: history is not rewritten, so the F2 commit message carries the correction about the refusing-seam row.

F2, F4 and F5 are one commit; `_lym/landing/summary.md` is edited and not committed.
`lym gate test --tier 1 ./internal/hubreconcile` passed after the fixes, and both `EnsureOrWarn` rows ran.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency, LOW, the summary's timings predate the restored walk: fixed differently.
  A rerun passed, but under a load average of 31 on 12 cores, which cannot be compared with the earlier figures.
  The summary now says the after figures predate the restored walk.
- fix-consistency, LOW, the `f2ebdb2c1` body counts three removed retry walks: fixed, by an empty commit that states two.
- fix-proposal, LOW, the timings predate the restored walk: duplicate of the first fix-consistency finding, same disposition.
- fix-affected-set, LOW, the timings predate the restored walk: duplicate, same disposition.
- fix-affected-set, LOW, the retry-walk count: duplicate of the second fix-consistency finding, fixed.

Two lenses also noted, outside their findings, that `docs/research/no-fake-other-modules/fabric-batten.md` still names `recordNotices` and `recordBoardSyncs`.
The reviewed change made those names stale, not this round's fixes, and nothing was changed there.

## Disputed

None.

## Deferred

None.

## Rulings

None.
