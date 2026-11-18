```yaml
lens: fix-affected-set
outcome: reported
```

Task: check whether the F1 and F2 fixes cover every member of the sets their premises rest on.

**F1 (dead-loop stop writes no interrupted record):** no findings.
- `loopExitStop` has two callers, `deadLoopStop` (`loopwait.go:590`) and `loopExitedEarlyStop` (`loopwait.go:610`).
- `loopExitedEarlyStop` passes a `LoopPIDRecord{LoopID}` with no child and no `TraceID`, so the new record guard skips it, which is correct.
- The teardown mark is excluded by the `rec.LoopID != TeardownLoopID` check.
- `newestInterruptedTraceID` has one caller, `reconcileInterruptedStep` (`loop.go:471`), and that function has one caller (`loop.go:239`).
- `interruptedRecord` now has two writers (`loop.go:296` and `loopwait.go:682`), and both write the same shape.

**F2 (unreadable registry entry hides a live holder):** no findings.
- `SessionHolder` has three consumers: `stepstop.go:135` (error reads held, good), `relaunch.go:37` (already refuses on a lookup error) and `Run.SessionHolder` (`run.go:1098`).
- `Claude` is the only `SessionHolderFinder` implementation outside test fakes. `claudeengine/resume.go:120` is the only registry lookup, and the fix covers both the `ReadErr` and `DecodeErr` branches.
- `Run.SessionHolder` has one non-test caller, `orchcli/start.go:203`. It already records the lookup error in `HolderErr` when no holder is found, so the new error reaches it as a recorded diagnostic, not a silent skip.

**Left unchecked:** the `ReadErr` branch of `sessionHolder` has no test row. Only the undecodable-entry row was added. This is a test gap, not a defect, and I did not weigh it against the rubric.

Tool calls:
1. Grep for the F1 sites (`loopExitStop`, `interruptedRecord`, `newestInterruptedTraceID`, `reconcileInterruptedStep`) and the F2 sites (`SessionHolder` and `SessionHolderFinder`) across `internal`. My context held the edited files but not their callers.
2. Read `orchcli/start.go:195-222` and grepped for `.SessionHolder()` callers. My context did not show the one external consumer of `Run.SessionHolder`.
