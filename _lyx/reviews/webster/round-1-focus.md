---
round: 1
focus:
  - "internal/fabricengine/scratchlink.go: does the migration of an existing real `.scratch` directory into the weft-side `.lyx` target lose, overwrite or leave behind files, and does a rerun converge to the same state?"
  - "The scratch link's skip in `Unwire`, `Remove` and `applyStaleRemoval`: does it drop only the name `.scratch` at a root anchor, and never a nested `.scratch` or another link?"
  - "`lyx fabric remove`: is the root `.scratch` link unlinked through the gate before the warp worktree is removed, and is the target's content left intact on every path including failures?"
  - "Reconcile: is the step run on every managed pair, are both outcome strings appended to the pair's Detail, and do the wiring verbs log the left outcome through `logger.Warn`?"
  - "Created or restored targets are recorded as `dir_created`: does the mutation oracle's omission direction cover them, and does a test pin it?"
  - "Windows and non-Windows link handling: do all link operations go through `internal/fslink` (`CreateDirLink`), and does the `fabric-windows` check join as card 4 states?"
  - "Test economy: do the new tests in `scratchlink_integration_test.go` and the cleanup tests duplicate coverage across cards, and does every behavior the diff adds have a test?"
  - "Declared PATTERN changes and docs: does the diff land each declared `PATTERN.md` / `pattern/` entry, and the `doc.go` updates for the changed observable CLI behavior?"
---

Seed focus from the plan's four cards and shared decisions.
