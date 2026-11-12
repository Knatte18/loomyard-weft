---
round: 3
focus:
  - "D4 in decision-record.md: does each map's build lock, like its write lock, come from lock.TrackedFileLockPath under the checkout's .lym/locks/ mirror, as PATTERN-lym-files requires for a tracked _lym file, with only the exit record never-tracked under .lym/testcover/? Check the Constraints PATTERN-lym-files bullet says the same."
  - "D9 in decision-record.md: runTestMap in internal/gatecli/testmap.go calls registerUpkeepPass only for --upkeep, and pairteardown's endOwnedProcesses reaches only registered records. Does the record make every lym gate testmap call in a task worktree, a named-package rebuild included, end with its pair's teardown, without blocking or being blocked by the prime's pass?"
  - "D4 push edge: a lym gate testmap call in a task pair spawns a detached push that carries the code branch. PATTERN-owner-machine (as declared changed) says a loom push or lym gate testmap's push in a task pair reconciles run-scope origin first. Does D4 state that reconcile and its bound, and do the PATTERN-pair-remote constraint and the acceptance criteria match?"
  - "D5 and fabric's weft pull: does the record say how a weft pull meets uncommitted _lym/testcover/ paths, from a running pass or a killed call, given weftDirtyStop and ErrReplayDirty in internal/fabricengine/pull.go, with a stated bound and an acceptance criterion?"
  - "D7: when a pass leaves a current map unwritten, does the record say what a package job and the redundancy step's map age and drift (internal/gatecli/redundancy.go) read, so none writes old file hashes under a new commit or reports staleness the plan ruled out?"
  - "D11 import: does the record state how the import avoids overwriting a map a live pass just wrote, with its lock and bound? Is the commit it names for the bolt maps the one the maps record?"
  - "D1, D4 and the Open risks: do the writer's and readers' path sources agree, and does the record carry no Open-risks line that an Added-after-Discussion entry already settles?"
---

Each entry names a site and the question to settle; none caps severity or states a verdict.
