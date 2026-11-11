Findings from the checks: three set members were not fixed.

1. **LOW** — `internal/shedverbs/doc.go:217` says "Spec.Loop.OnStart runs once in the loop." The fix changed only lines 228–229, so this opening line is still stale. The keeper also runs the hook, and line 219 already says so. The line still reads fine beside the fix to the `OnStart` comment in `loop.go`.

2. **LOW** — `internal/shedverbs/loop.go:185` (the `runLoop` doc) says "LoopSpec.OnStart runs once after the start is logged and before the first step." The fix to the field comment did not reach this restatement. It describes only the loop process's own run, so it is accurate but incomplete, and it does not mention the keeper's run.

3. **LOW** — Loop-process wording stays stale in code the fix did not touch. `internal/shedrun/paths.go:551` says "the stdout and stderr of the detached loop and its keeper", which is fine. `internal/shedverbs/loop.go:73` says `Executable` is "the program a child runs, and the program the detached loop itself is". The keeper also runs this executable, so it is now incomplete. `internal/shedverbs/loopenvelope.go:196` says `LoopID` is "the id the detached loop process was started with". The keeper also carries the id, so this is incomplete too. All three are cosmetic, and the review named only the `Spawn` comment.

4. **LOW** — F3 coverage is partial. `DetachedLoopID` has two callers outside the tests: `internal/shedcli/cli.go:205` and `internal/loomcli/cli.go:271`. The new keeper-flag row drives only the `shed step` path. The existing "the loom surface" row runs without the keeper flag. A revert of `DetachedLoopID` to the detached flag alone still passes through the `loom step` path. The review named only the `shedcli`/`loomcli` pair as the premise, and a `loom step --loop-keeper` row would cover it.

No findings for F4: `deadLoopStop` has three call sites (`loopwait.go:94`, `:99`, `:132`), and all go through the same function that now waits. F5 and F6 had no other stale restatements beyond the items above, apart from `loomcli/doc.go:518` and `battencli/arm.go:469`. Those describe the hook's use and already mention the keeper for `battencli`. I did not read `loomcli/doc.go:518` further, so I left it unchecked.

Tool calls:
1. grep for `OnStart` across `internal` — to list every site restating the hook's contract, which my context lacked.
2. grep for detached-loop, spawn and exit wording — to find stale descriptions of what `Spawn` and the loop start.
3. `sed` of `internal/loomcli/cli.go:265-285` plus a grep of the loomcli and shedcli tests for the loop flags — to check the second `DetachedLoopID` caller and whether tests cover it.
