```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the test-map refresh diff.

**Per function, case run, verdict**
- `countLandings` (`tidy.go:143`), first read of a record: it baselines and returns before the new add, so no landing is counted. Correct.
- `countLandings`, two fresh landings with the seams wired and `test_map_every` 0: the count still grows by 2 each time, with no cap. Harmless while 0, but flipping the key to 5 later starts a refresh at once. Finding 2.
- `countLandings`, a tidy agent in flight: it returns early, the landings stay uncounted, and they are counted after the tidy. Correct.
- `countLandings` on a beam's dobby (`StartTestMap` nil): nothing is added. Correct.
- `testMapStep`, `test_map_every` 0: it returns before reading `TestMapRunning` or starting anything. Correct.
- `testMapStep`, count below the threshold: it returns. Correct.
- `testMapStep`, count at the threshold with a live refresh: it keeps the count and starts none. After the live one ends, the next tick starts one. Correct.
- `testMapStep`, `TestMapRunning` or `StartTestMap` returning an error: it returns a wrapped error with the count kept, so the retry is correct. But the error aborts the whole tick. Finding 1.
- `testMapStep`, start succeeds and the final `UpdateState` fails: the refresh runs and is registered, but the count stays at or above the threshold. When that refresh ends, a second one starts at once for the same landings. LOW. The window is small, and the same shape exists in `spawnTidy`.
- `startTestMap`, `LymBinary` error: it returns the error before spawning. Correct. Under `go test` it refuses the test binary, as the doc says.
- `startTestMap`, `cmd.Start` fails: it logs a warning and returns the error. Correct.
- `startTestMap`, normal start: `proc.Detach` sets `Setsid`, so the child leads its own group. `ChildRecord` sets `PGID` to the pid. The goroutine reaps the child with `cmd.Wait`. Correct.
- `startTestMap`, child exits before `ChildRecord` reads its start time: `StartTime` is empty and the record file is `<pid>-.json`. `List` prunes it, because the process is gone. Correct.
- `startTestMap`, `Register` fails: it calls `EndProcess` and returns the `Register` error. The `EndProcess` error is only logged. Correct. But a persistent `Register` failure (for example an unwritable run-proc dir) spawns and kills a process on every tick. Finding 1 covers the repeat.
- `testMapRunning`: it matches `Role == RoleTestMap` among the `HubOwner` records. `List` already drops dead records, so a stale record is pruned and counts as not running. Correct. It uses `List`, which treats zombies as not live; a zombie child is reaped by the goroutine soon after, so that is fine.
- `LoadConfig` with `test_map_batch` below 1 or `test_map_every` negative: it refuses and names the way forward. Correct.
- `endDobbyRoles`, prime owner: it ends the dobby, then `RoleTestMap`. A beam owner ends the dobby only. A dobby-end error skips the refresh end and returns the error. This matches its test. Correct, apart from finding 3.
- `testMapStep` placement in `tick`: it sits inside `moving == ""`, after `tidyStep`. Correct.

**Findings**
1. MEDIUM, design. `internal/dobbyengine/loop.go:648` and `testmap.go:33-40`. Any `testMapStep` error returns from `tick`, so `landBeam`, `announceLandings` and the `Deps.Tick` hook are skipped. On the prime that hook carries the hibernate carrier and the Slack relay (`dobbydeps.go:213`). A persistent failure of an optional refresh, such as an unresolvable binary, an unreadable run-proc dir or a failing `Register`, stalls those every tick. A failing `Register` also spawns and kills a process each tick. The step should surface the failure without aborting the tick, for example by logging once per episode and carrying on. `tidyStep` has the same shape, but that does not make this correct.
2. LOW, design. `tidy.go:143-145`. The count is added whenever `StartTestMap` is wired, even with `test_map_every` 0. It is unbounded, and enabling the key later fires a refresh immediately on a stale count. Add only when `TestMapEvery > 0`, or reset the count while it is 0.
3. MEDIUM, design. `orchcli/stop.go:57-70`. `beamDown` returns before `endDobby` when the dobby is not live, so `endDobbyRoles` never runs. If the prime's dobby died and a refresh is still running, `lym beam stop` of the prime leaves that refresh alive. The Goal says the child is "ended on stop". The doc covers only a dobby that dies without a stop; this case is a stop with the dobby already gone. Ending `RoleTestMap` for the prime should not depend on the dobby being live.

Tool calls: none; the inherited context held everything the lens needed.
