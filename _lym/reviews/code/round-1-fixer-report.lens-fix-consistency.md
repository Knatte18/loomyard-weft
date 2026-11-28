```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency check of the round-1 fix change set against its surroundings.

1. **MEDIUM**: `doc.go` "the prime's push summary reports every live dobby's wait on origin", with the `hubBeamCatchUpWaits` comment and the `summary.md` bullet "A stopped dobby's record is left out, since nothing retries or clears it".
   - A live dobby's record can also be one that nothing retries or clears. `catchUpOwed` lives only in memory.
   - `recordStart` (`loop.go`) returns early on a re-exec and owes no catch-up. `State.CatchUp` is cleared only in `catchUpSucceeded`.
   - So a dobby that re-execs during an origin wait keeps the record until some other trigger owes a catch-up.
   - The summary then reports it with growing minutes and holds back the all-clear. This now applies to every beam dobby as well as the prime.

2. **LOW**: `doc.go`, catch-up section, "The dobby owes the catch-up ... when a pass exits PassExitOriginWait or PassExitOwnedElsewhere".
   - This is no longer true for a first pass. `oweCatchUpForStart` returns false without owing on a repeat `PassExitOriginWait` after a successful catch-up.
   - The passes section says such an exit is a refused start but does not say it owes nothing.

3. **LOW**: the `internal/battencli/run_test.go` header comment (about lines 243-246).
   - It names the origin-wait exit for an unreachable origin. For a registration refusal it says only "passes its way forward", and for any other failure only "names its error and the re-run".
   - Both of those rows now expect `PassExitCatchUpRefused`, a distinct exit that the comment does not mention. The twin comment in `owner_test.go` was updated.

4. **LOW**: `pushsummary.go`, the heading changed to "waiting on the network:".
   - Its twin, the all-clear "every branch is pushed to origin and no run waits on the network" (and `allClear` in the test), still speaks only of runs.
   - After a summary that listed only a dobby's or a beam's catch-up, the all-clear says nothing about the thing it reported.

Tool calls:
1. `git grep` for exit codes, push-summary and refusal wording outside the files already read: my context held only the changed files and earlier narrow greps.
2. `sed -n` on `run_test.go` lines 236-252, plus `git grep` for `push_notice_after_min` and all-clear wording in docs and config: the `run_test.go` header and the config template were not in context.
