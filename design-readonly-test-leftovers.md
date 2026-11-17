# Sweep read-only leftovers a killed test leaves under the scratch root

`readonly-test-leftovers` · Task · bug, cleanup, testkit, webster · running · Code-Burler-Review

## Seen (orch, 2026-11-17)

A test that makes a directory read-only and restores it only in `t.Cleanup` leaves it read-only when the test binary is killed:
the memory cap of `lym gate redundant` killed `internal/websterengine`'s `TestRun_RefusesBeforeSpawn` mid-run,
and three read-only leftovers stayed under `/tmp/lym-1000`.
The cleanup sweep could not unlink them, so `internal/cleanupcli`'s `TestCleanupVerb_ScratchSegmentSweepsTheAnchorOfASubpathHub`
failed in `prompt-engine-per-launch`'s verify gate on machine state, not on its change.

The pattern is not one test's: `grep -rn 'Chmod(.*0o555' --include=*_test.go internal` finds it in several packages
(webster, fabric, config, shuttle, machine, vscode, friction).

## Idea

- The scratch sweep of the cleanup verb makes a leftover tree owner-writable before it removes it,
  so a killed test's read-only directory is swept like any other leftover.
- A test that needs a read-only directory takes it from one testkit helper that registers the restore,
  so the pattern lives in one place.
