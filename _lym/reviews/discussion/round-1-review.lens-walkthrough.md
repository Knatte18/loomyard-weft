```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each Decision's mechanism run end to end on one concrete case, checked against its own stated purpose.

**Decisions walked**

- **D1**, a re-exec while `lym batten overview` probes the lock for microseconds. `runDobby` → `placeDobby` → `hubDobbyDeps` → `Run` takes the lock within its grace → registration seam → `recordStart`. Purpose met for the probe case.
- **D2**, a re-exec with `state.json` pid equal to `os.Getpid()`. `Run` reads that pid before taking the lock and picks the ten-second wait; a second dobby spawned by `EnsureLive` reads someone else's pid and keeps `ProbeGrace`. The re-exec decision is made before the winner's `recordStart` overwrites the pid. Purpose met.
- **D3**, a re-exec onto a new binary whose `hubDobbyDeps` (or `placeDobby`) fails. It exits before `Run`, with no `Log` seam yet, so the error envelope goes to stdout only. Purpose missed (F1).
- **D4**, a tidy in flight at the re-exec. `recordStart` skips the start branch and `tidyStep` → `settleTidy` acts on `State.Tidy`. Purpose met, but the stated reason does not match the code (F3).
- **D5**, a fresh spawn: stdout and stderr go to `dobby.out`, and `Log` opens `geom.LogPath` once per line with rotation. A refused fresh start's envelope lands in `dobby.out` and its line in `dobby.log`. Purpose met.
- **D6**, a run that goes from `running` to `halted`, then a re-exec. `RunView` already carries `State` and `Producer`, and the start heartbeat lists every state. `FailingPassCount` sits in the persistent state record, so the first failure of an episode is not logged again after a re-exec. Purpose met.
- **D7**, the Land row of a `lym batten run` pass on machine A: the shed is told A, and the Land Done entry carries `machine: A`. Purpose met.
- **D8**, A lands X, B pulls the record. `DobbyLandings` on B skips X, A counts it, and A's `LandedSinceTidy` survives a dobby exit (no writer outside `tidy.go` clears it). Purpose met.
- **D9**, the hub's `finalize: {tiers: [0,1,2], scope: all}` with no `adopt`. `MigrateConfig` gains a line, `Ensure` writes it once per build stamp, and the next pass changes nothing. Purpose met.
- **D10**, `lym adopt root`. The spawner truncates `suite.log`, the detached `lym adopt check` runs the gate in the prime, and the record and per-tier posts follow. Purpose met in steady state; the first moments after the spawn read the previous record (F2).
- **D11**, `lym beam status` right after `lym adopt`, and a dead-pid record. Dead pid shows `ended without a verdict`; met. Right after a spawn it shows the old verdict (F2).

**Findings**

- **F1 — MEDIUM, design, Decision 3 (and Decision 5; Acceptance "Each exit path of Decision 3 writes its line")**
  - `runDobby` exits before `Run` when `placeDobby` refuses or `hubDobbyDeps` fails, and neither exit is in Decision 3's list.
  - After Decision 5 moves stdout to `dobby.out`, those exits leave nothing in `dobby.log` and post nothing to the orch.
  - A re-exec onto a newly deployed binary is exactly when new wiring code runs first, so this is the incident's own scenario, and the Goal "never ends silently" fails there.
  - Evidence: `internal/beamcli/cli.go:372-389`. `placeDobby` and `hubDobbyDeps` return `output.Err(out, …)` before `Run`. `hubDobbyDeps` builds the `Log` seam itself, so a failure there leaves no seam (`internal/beamcli/dobbydeps.go:hubDobbyDeps`, `Log: …Fprintln(logOut…)`).
- **F2 — LOW, design, Decision 10 / Decision 11**
  - `check.json` is "rewritten fresh at each start" by the check process, but between `lym adopt`'s spawn and the child's first record write, `lym beam status` shows the previous check's record, for example a green verdict from an earlier adopt.
  - Nothing says the spawner clears or seeds the record before it returns the envelope.
  - Evidence: `internal/adoptcli/deps.go:startSuite`. The spawner alone truncates `suite.log` (`os.Create`) before `command.Start`, and the record is written only by the child, per Decision 10.
- **F3 — NIT, consistency, Decision 4**
  - "A re-exec reads its own pid in the record and keeps `State.Tidy`" ties the keep to the pid check, but every start keeps `State.Tidy`.
  - `recordStart` writes only `PID` and `StartTime`, and `tidyStep` settles any `State.Tidy` whatever the pid, so a second dobby that wins the lock (Decision 2's bound) settles the tidy too.
  - Evidence: `internal/dobbyengine/loop.go:190-204` (`recordStart`), `internal/dobbyengine/tidy.go:146-150` (`tidyStep` → `settleTidy`).

**Tool calls**

1. `grep` of `RunView` in `internal/dobbyengine/passes.go` and its failing-pass tracking. I needed to confirm that Decision 6's heartbeat and change lines have their state and producer in hand, and that the failing-pass count persists across a re-exec; the manifest did not cover `RunView` or `passes.go` beyond `logLine`.
