Task: check that the fix covers every member of each finding's set (stale doc claims, callers, flag users).

Findings:

- **LOW** — `internal/shedverbs/loopwait.go:70-71` (`runUntilStop` doc), F5's set.
  - The fix rewrote the `LoopSpec.Spawn` doc in `loop.go`, but the same stale wording remains here.
  - Line 70 says it starts "the detached loop first" and line 71 says "the detached loop receives with the flags that select it".
  - Line 114 of the same file spawns the keeper with `--loop-keeper=<id>`, and `loopwait.go:1` has the same wording.
  - `shedrun/paths.go:551` already says "the detached loop and its keeper".
  - A reader of these docs would take the waiter to start the loop process itself.

Searches that turned up no further findings:

- **F6, OnStart/OnExit claims:** the only claims are `loop.go:88-93`, `loop.go:184` and `doc.go:217-228`, and all three agree with the fix. `loomcli/doc.go:518` and `battencli/arm.go:469` describe `OnStart` registering the keeper and are consistent.
- **F4, `deadLoopStop` callers:** all three call sites in `loopwait.go` (94, 99, 132) pass the new argument, and it has no other callers.
- **F3, flag users:** `DetachedLoopID` is read in `shedcli/cli.go:205`, `loomcli/cli.go:271` and `step.go:351`. The keeper flag is covered in all three through the helper, and the arming test drives the real binary.
- **F9, "now reads" clause:** the only other place that asserts it is `loopwait_test.go:531`, which still matches (A→B).

Calls made:
1. `grep` for `OnStart` and the "detached loop" wording across `internal` and `docs`, to find other members of the F5 and F6 sets that my context lacked.
2. `grep` for the flags, `DetachedLoopID`, `deadLoopStop(` and "now reads" across the repo, to find the callers and flag users that my context lacked.
