```yaml
lens: code-correctness
outcome: reported
```

Task: review the correctness of the code the diff changes (lens code-correctness).

**Per-function list**

- `drivePasses` filter, done run with no flight: it is dropped from `runs` and `d.runs`. `recordOwed` then deletes its `Runs` entry, even when `owesAll` is set, because the loop only owes listed runs. **Correct.** `h.tick(true)` in the new test confirms the fresh-start case.
- `drivePasses` filter, done run with a flight: it is kept, so it keeps its slot in `runsInFlight`, its owed state, and `busy = true` through the `InFlight` branch. **Correct.**
- Exit read and drop in the same tick: `collectExits` runs before the filter. It deletes the flight and `passExited` writes the entry. The filter then drops the run and `recordOwed` deletes that entry in the same tick, so the slot frees and the entry goes together. **Correct.** The second new test shows this.
- First-pass flight (`first` set, slug in `d.flights`) whose run already shows `done`: the run stays listed and `runsInFlight` counts it once, through `listed`. **Correct.**
- `slices.DeleteFunc` on the `deps.Runs()` slice: it shifts elements in place and zeroes the tail. The test fake hands over a copy (`passes_test.go:135`). The real `parentRuns` builds a fresh `runs` slice (`beamcli/dobbydeps.go:292`). So no caller's backing array is mutated. **Correct.**
- `originOwed` reads `deps.Runs()` unfiltered, which is right because it only reads `OriginOwed` for a run whose pass just exited. A run that finished done has its `OriginOwed` read as before. **Correct.**
- `startReady` and `runsInFlight` read only the filtered `d.runs`. Nothing else reads the unfiltered list, which `grep` confirmed. **Correct.**
- `d.owesAll` with every listed run done: nothing is owed, nothing is planned, and `busy` stays false. A run later reopened by a hand pass has no owed entry, so it gets a continue pass if it sits at `Run-Shed`. The decision record accepts this under "Open risks". **Correct as specified.**
- Tests, `done runs take no slot…`: with `maxRuns` 1, two done runs and `owesAll`, the task `a` starts, the only spawn is `a`, and the state holds no entry. The seeded `x` entry is removed. **Correct.**
- Tests, `a done run keeps its slot…`: with the pass held in flight and the run turned done, `a` is not spawned. After the exit, `a` is spawned once and the `r` entry is gone. **Correct.**

**Findings**

1. **LOW, design**, `internal/dobbyengine/passes.go:~137` (filter) with `triggerFor` (`wasCovered`, `childSeen`).
   - A run dropped as done leaves its `d.wasCovered` and `d.childSeen` memory behind. Only `liftPending` is pruned for unlisted slugs.
   - If a hand pass reopens that run, a stale `wasCovered[slug]=true` with the run now uncovered raises a spurious `liftPending` and a full pass. A stale `childSeen` can also give a spurious `ChildWrite`.
   - The same leak already exists for any run that goes unlisted, such as a pruned one. The diff makes it reachable for every finished run that is later reopened.
   - The effect is at most one extra full or continue pass, and it self-heals. Severity is LOW and only borderline a finding.

No other defect found.

**Tool calls (1 of 6)**
- `grep` of the test fake's `Runs` and of `parentRuns` in `beamcli/dobbydeps.go`: to check that `slices.DeleteFunc` does not mutate a shared backing array. The manifest lacked both.
