```yaml
lens: fix-claims
outcome: reported
```

Lens fix-claims: I checked every sentence the fix added or rewrote that states a fact, bound or behaviour against the code at HEAD.

**MEDIUM — card 7, runWorker step 3, the rewritten follow-verdict sentence.**
The new sentence says the verdict "is reached when a worker that ended released `adopt.lock` while its record still reads `running` and its pid still matches."
After the same round's card 3 change, that path no longer reaches it.
- Card 3 now makes `WorkerAlive` read `proc.IsRunning`.
- `internal/proc/proc_linux.go:31-37`: `IsRunning` answers false for a zombie (`fields[0] != "Z"`), so an exited but unreaped worker reads gone.
- `ReadState` then returns `interrupted` and the verdict is `resume`, not `follow`.
- The lock is released only once the process has exited.

The verdict is still reachable, so its reaching row stands; only the stated path is wrong. Two real paths reach it:
- A record without start ticks, where `WorkerAlive` reads alive on the pid alone and that pid has been reused (Windows, where `StartTime` returns false: `internal/proc/proc_windows.go`).
- Any running record whose pid and ticks match a live process that does not hold the lock, which is what the planned test row builds.

The sentence should name a reachable path, or simply say the verdict is reached whenever the record's worker reads alive while this worker holds the lock.

These claims hold:
- Card 6: `liftOwnerPauses` is the only writer of `PauseLifted` (`adopt.go:1005-1017`).
- Card 6: `callerLabel` is read only by the `Lift` seams in `deps.go:91` (`hubDeps`, card 6) and `green.go:253` (`hubGreenDeps`, card 4).
- Card 6: `runningHereRun` is set only inside `holdTargets` (`holds.go:131`, use at `:175`); its other reader is the TestAdopt row card 6 rewrites.
- Card 6: the `movedRuns` signature takes the owners.
- Card 7: `collect` exists (`adopt.go:1035`).
- Card 1: the hold-caller sentence at `passes.go:108` is part of `planPass`'s doc comment (`func planPass` at `:116`).
- Card 2: the fake starter appends `"start"` to `h.calls` per session start (`start_test.go:52-54`), and `h.spawns` counts watcher spawns.
- Card 3: on Linux `IsRunning` reads a zombie as gone.
- Card 4: no `_test.go` asserts the old `OwnedElsewhereError` or fix-loop text (the parent's earlier grep found none).
- Cards 6 and 7, `TestAdoptIntegration_SecondCloneTakesOver`: the adopt back's live-pair stop, the `--drop-local` refusal naming `--keep-local` and the `--keep-local` prime record match `adopt_integration_test.go:172-191`.

Tool calls:
1. One Bash call reading `liftOwnerPauses` (`adopt.go:1005-1020`), the `Lift` seam in `green.go:245-256`, the enclosing function of `holds.go:175`, the fake starter in `start_test.go:40-60`, and `planPass` and `collect` locations. My context did not show which function holds each line, or what the starter records as a call.
