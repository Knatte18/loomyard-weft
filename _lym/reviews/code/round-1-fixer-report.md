# Fixer-report, round 1

## Changes

- F3, F4, F5, F6 are fixed, one commit each, except that F3 and F5 share a file and a commit.
- F4: `internal/fabricengine/doc.go` and `internal/fabricengine/destroy_test.go` name `containmentFailure` in place of the deleted `refuseUncontainedPath` (03c24559).
- F6: `hubRel` moved from `retiredlaunchers.go` to `hubworkspace.go`, unchanged (db1da40e).
- F3: row `SymlinkAtTheContainerIsKeptAndNotFollowed` added to `TestSweepRetiredLaunchers`, a link at `_launchers` to an outside directory holding `ide.sh` and `sentinel.txt`; the harness now asserts every outside file untouched (17393100).
- F5: `.cmd` launcher scripts seeded in `fabricWrittenTree` and listed in `fabricWrittenRemoved`; the self-check widened it from `run.cmd` alone to all four `.cmd` names (17393100, 2841a1cc).
- Gate: `lym gate test ./internal/fabricengine` (tier 0) passes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW: the F5 fix pinned only `run.cmd` of the four `.cmd` names. Fixed as proposed by the fork, in 2841a1cc.

## Disputed

None.

## Deferred

- F2 (MEDIUM, refusal-envelope shape no longer tested): not done.
  The reviewer's proposed trigger, a pair whose warp worktree path is a link out of the hub, does not refuse.
  Run through `lym fabric remove --force`, the removal succeeded with exit 0, because the gate keeps the final path component unresolved and lets a link be removed.
  Two other triggers, a retargeted `.lym` or `_lym` junction through `remove --force` and through `unwire --apply`, did not refuse either: the scan of wired junctions skips a drifted link and the verb succeeds.
  No surviving test asserts the positive `refusal` object through the CLI, and a refusal needs an intermediate link below the container, which the hub the scenario uses (anchor ".") does not offer.
  The change is reverted and nothing of it is committed.
  A test needs either a hub with a subpath anchor or a fabricengine test seam, both beyond this round's write surface.

## Rulings

- F1 (parent decision, creel #1193): keep the reserved `_launchers` name in `internal/fabricengine/junctionnames.go`, and refuse the finding as a correction.
  The code already holds what the ruling asks, so nothing changed.
  Not flagged to the parent, since the ruling is carried out as given.
