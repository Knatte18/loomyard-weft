# Name the dirty paths when a landing fixer halts on a dirty tree

A Parent-Align or Finalize merge fixer that left the task worktree dirty after its last turn halted with `the fixer session of <row> ended without a usable turn (dirty)`.
The reason named no path, so the operator could not tell whether the branch needed work.
The halt reason now lists the uncommitted or untracked paths and says to commit or remove them before `lym loom resume`.

## What changes

- `fixerOutcome` in `internal/landingshed/fixerrun.go` carries the dirty paths the last turn left, set only for the dirty end.
- `haltReason` gives the dirty end its own reason: the paths, capped by `listShown` like every other dirty-tree reason in the package, the session log and the report path, with the way forward `commit or remove them on the task branch, then lym loom resume`.
  The timeout and no-report ends keep the old wording.
- `TestFixerRun_Outcomes`'s dirty row asserts the path and the new way forward appear and the old generic wording does not.
- The package doc's list of fixer halts names the dirty-tree halt and says its reason names the paths.

## Design decisions

- The issue's first bullet, to treat a passed gate plus a finished fixer turn as a clean end, is already handled on main by `b5e0ed26` (`Hold writer and fixer sessions until their worktree is clean`).
  The incident logged `end=dirty spent=1 cap=2`: under that commit, a dirty tree with a turn left re-prompts the fixer with the paths on a charged turn, and only a dirty tree after the last turn halts, at `spent == cap`.
  The incident predates that commit.
- A dirty tree after the last turn stays a halt rather than counting as clean.
  The landing gate's verify refuses a dirty tree, and Parent-Align and Finalize must never merge over or land uncommitted work, so the passed gate the issue cites cannot stand for a tree that changed after the gate.

## To check by hand

- The incident's dirty paths are not recorded, so their source is unknown.
  An untracked file the fixer did not make, such as the stale `CLAUDE.local.md` that board note `heddle-stale-root-claude-file` describes, would still use up the re-prompt turn and halt.
  The new reason names such a path, which shows whether that note is the cause.
