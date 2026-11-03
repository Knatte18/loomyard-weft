```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the F1/F2 fixes cover every member of their sets.

F1 callers. All `taskWorktreeLocation` and `taskWorktreeLocator` callers (`arm.go:1023`, `dobbywire.go:19`, `cancelcmd.go:396,411`, `readopt.go:78`, `goto.go:19`, `dobbynotice.go:32`, `wire.go:244`) now reach `presentTaskWorktreePath`, which uses `taskWorktreePresent`. They all refuse a stray directory by name.

Finding: LOW, `internal/battencli/wire.go:250-257` (the locator's cached branch).
- Once a resolution has succeeded, the cached branch tests `os.Stat` only. A stray directory at that point passes, so the "stray directory refused by name" behavior does not hold there.
- This is deliberate. `stepTaskWorktreeSeams_RunGitOnlyUntilTheFirstResolution` hides `.git` and requires the cached answer, and the locator doc says so.
- The package doc at `cli.go:69` says "every batten verb reads it so", which the cached branch does not match.
- Fix, if wanted: add "after the first resolution" to the `cli.go:69` or locator wording.

Other stat-only sites that read a stray directory as present:
- `beamland.go:140`: `os.Stat(WorktreePath)`.
- `overview.go:290`: `os.Stat(...).IsDir()`.
- `refusal.go:122`: stat of the parent's `.git`.
- These are not callers of `taskWorktreeLocation`, so F1's fix did not have to cover them, and `cli.go:69` still claims "every batten verb". I did not read their code, so I can't say whether they diverge in practice.
- Fix, if wanted: soften `cli.go:69` to name the callers that use `taskWorktreePresent`, or route these sites through it.

F2 set: the `hubStartGuard` seams. `startguard.go:88` is the only `ReachedParent` wiring (`startguard.go:36,142` is the seam and its use). The new subtest covers it, and I found no other unpinned seam in that finding's key.

Unchecked: none.

Calls:
1. grep in `internal/battencli` for `WorktreePath(`, `taskWorktreeLocation(`, `taskWorktreeLocator(`, "every batten verb", "one answer". I ran it to list every caller and stat-only site, and my context had none of them.
