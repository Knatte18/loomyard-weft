```yaml
lens: walkthrough
outcome: reported
```

Task: the walkthrough lens. I ran each decision's mechanism step by step on one concrete case.

**Walkthrough list**

- **D1** (a fresh beam dobby; its worktree is then removed and the next probe tick runs): purpose not met. The exit line has nowhere to land (F1).
- **D2** (a fresh dobby after a reboot whose pid equals the stale record's pid; also a dobby that only re-execs from the first deploy on): purpose not met. `recordStart` still keys on pid alone (F2).
- **D3** (a failed exec on an unchanged identity, then a refused re-exec start, then a wiring failure after a re-exec): purpose met. `Check` records the failed identity and does not retry it, so there is one notice. The worktree-gone exit line is the exception, covered under F1.
- **D4** (a re-exec while `State.Tidy` is set): purpose met as the record states it. A start rewrites only pid, time and ticks, and the tidy step settles by record.
- **D5** (a dobby spawned by the pre-change binary that then re-execs onto the shipping binary): purpose not met. Its inherited fd 1 and 2 still point at `dobby.log` (F3).
- **D6** (a run going halted between two ticks, then 10 min of idle ticks): purpose met. The loop wakes at least once a minute, so a heartbeat comes within 11 min.
- **D7** (a batten pass's Land row on owning machine A): purpose met. The pass process's wiring tells `machinename.Name()`, and the history entry carries A.
- **D8** (run X landed on A; machine B pulls the record, and B later adopts root): purpose met. `DobbyLandings` on B skips X, and X never enters B's `CountedLandings`.
- **D9** (a hub `gate.yaml` tuning `finalize` to `[0,1,2]`, no `adopt`, stale stamp, then `lym adopt`): purpose met on the check path. The check's `Ensure` runs the gate row's `Migrate`, then `loadGate` reads `adopt` as `[0,1,2]`.
- **D10** (`lym adopt` twice in a row over a previous green `check.json`): purpose met within the stated bound. The second child loses the lock and exits, and its envelope names a check that never ran.
- **D11** (a running record whose pid was reused after a reboot): purpose met. The pid-plus-ticks test reads it as ended without a verdict.

**Findings**

F1 — MEDIUM, design. Location: Decision 1 (worktree-gone exit), Decision 3's line list, and the AC "A beam dobby whose worktree is removed exits at its next probe tick and writes the exit line".
- Summary: a beam dobby's `dobby.log` lives inside the beam worktree that was removed, so the exit line cannot be written there.
- Mechanism: `Geometry.LogPath` is `<HubPath>/<worktree>/<anchorRel>/.lym/beam/dobby.log`.
  The per-line append of Decision 5 then either fails because the directory is gone, or recreates part of the removed worktree's path if the seam creates directories.
  The record names no other destination for this line, and the AC asserts a write the geometry cannot take.
- Evidence: `internal/hubgeom/dobbygeom.go:26`, `:47`, `:57`.

F2 — MEDIUM, design. Location: Decision 2 and Decision 4, with the AC "a fresh dobby whose pid matches the record's but whose start ticks differ waits only `lym.ProbeGrace`…".
- Summary: Decision 2 applies its pid-plus-start-ticks re-exec test only to the lock wait. `recordStart`'s own start-or-re-exec decision still compares the pid alone.
- Consequence 1: a fresh dobby whose pid matches a stale record gets the short wait, as intended. But `recordStart` then returns early as a re-exec, so `owesAll` stays false and the record's start time and start ticks are never rewritten.
- Consequence 2: every later re-exec of that dobby fails the ticks test and gets only `lock.ProbeGrace`, the incident's failure mode.
- Consequence 3: a legacy record with no ticks is never upgraded through re-execs, because a re-exec writes nothing. So the pid-reuse protection the ticks exist for never arrives for a dobby that only re-execs.
- The record should say `recordStart` uses the same test and writes the ticks when they are absent.
- Evidence: `internal/dobbyengine/loop.go:190-203`, where `recordStart` returns nil on `record.PID == os.Getpid()` and writes pid and start time only on a start.

F3 — MEDIUM, design. Location: Decision 5 ("no raw process output reaches the file") and the AC on rotation.
- Summary: a dobby spawned by a binary from before this change has `dobby.log` as its fd 1 and 2, and `syscall.Exec` keeps them across every re-exec.
- Consequence 1: such a dobby goes on writing raw stdout and stderr (logger lines, a panic, its final envelope) into `dobby.log`.
- Consequence 2: after the first rotation those writes follow the renamed inode into `dobby.log.1`, and once the next rotation replaces `.1` they land in an unlinked file. A panic text is then lost.
- Under Decision 1 the dobby survives deploys by re-exec and is never respawned, so this lasts indefinitely. The record states no bound for it and no step that moves the inherited fds, such as reopening fd 1 and 2 onto `dobby.out` at start.
- Evidence: `internal/dobbyengine/start.go:267-277`, where `cmd.Stdout = cmd.Stderr = dobby.log`, and `internal/selfexec/selfexec.go:76`, where the exec passes the same process's fds.

Tool calls:
1. `loop.go` `recordStart` tail, `SpawnDetached` and `selfexec.Check`: the manifest held `Run` but not the end of `recordStart`, the spawn's fd wiring or `Check`'s exec and retry.
2. `LandedSinceTidy` and `DobbyLandings`: to walk D8's counting, which the manifest did not hold.
3. `tidy.go:105-135`: to see how landings join the count (`CountedLandings`, baselining).
4. `dobbygeom.go:55-75` and `NotifyFromScoped`: to confirm where `LogPath` lies for a beam, and the notice signature D3 relies on.
