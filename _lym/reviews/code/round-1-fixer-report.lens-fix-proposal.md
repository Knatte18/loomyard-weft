```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: challenge the review's proposals that the fixer carried out as proposed (F1, F3, F4, F5, F6, F9, F10), plus F2's own variant. List of those fixed as proposed: F1, F3, F4, F5, F6, F9, F10.

**LOW. F5, `internal/gatecli/tierrun.go` (the new `!errors.As(runErr, &exitErr)` refusal)**
- The refusal fires on any non-`*exec.ExitError` from a tier with `memory_mb`, not only a cap that could not be set.
- Case: the counter's output writer fails, for example a closed stdout pipe. `lineReader.Write` returns the `passThrough` error, `cmd.Wait` returns that copy error, and the tier is refused as "did not run under its memory_mb".
- Before the fix this was exit 1 with a cause in the log. The wording is also wrong for a failure that has nothing to do with the cap.
- Evidence: `runTiedChild` returns `waitErr` unchanged except for `ErrWaitDelay`.

**LOW. F2, `testcover.Toolchain.GuardKilled` / `startMemoryGuard` (`internal/gatecli/memoryguard.go`)**
- The killed-pid set only grows, and `RunAlone` tests only `cmd.Process.Pid` against it.
- The guard records every pid of a killed tree, descendants included. A later failing run whose pid the kernel recycled from one of those is reported as `GuardKilled`, so a genuine failure turns into a "not judged".
- It needs a pid wrap during one long redundancy or testmap call. It also only matters when `err != nil`, so it is rare.

**LOW. F2, `redundantmutants.go` `record` / `guarded`**
- A guard kill of a covering test that is not itself a candidate is dropped: no kill is recorded and no candidate is marked.
- The covering test then looks as if it did not kill that mutant. The candidate gets `Unique` mutants and is listed as refuted, falsely, because of a memory-pressure kill.
- The direction is conservative, since the test is kept, but the listing is wrong.
- Evidence: `guarded[jobs[i].test.Name]` only reaches `candidates[i].GuardKilled` when the name is a candidate (end-of-function loop).

**LOW. F1, `tierrun.go` `capKilled && stops.processCap == ""`**
- When the process cap and a real kernel OOM kill both happen in one run, the kernel kill's `memoryCap` line is dropped.
- `memoryStopped` stays set only through the process cap's flag. This is a race-only case and harmless.

No findings against F3, F4, F6, F9 or F10.
- F3: `exe.request.memoryGuarding` already carries the enclosing-run decision, so the guard is not doubled.
- F4: a killed binary leaves its tests with a run event and no result, which the experiment confirmed.

Further tool calls: none.
