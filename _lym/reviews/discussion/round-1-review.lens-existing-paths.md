```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths, tracing the code that already runs and that each Decision's state, file or invariant also meets.

**Paths traced, per Decision**

- D1 `registerDobby` behind the lock:
  - `runproc` record read by `lym beam stop` (`orchcli/cli.go:262` `EndRole`) and by `stopBeamDobby` (`loomcli/pawlwiring.go:285-290`): survives. A re-exec keeps its pid and the old image's record, so the waiting image can still be ended.
  - Pair teardown `endOwnedProcesses` / `EndSparing` (`pairteardown/teardown.go:596`) and its `battenAlive` read of "a live dobby record under the run's parent" (`teardown.go:611-629`): survives for a re-exec. For a fresh start it reads no owner for longer than today, because registration now waits for `hubDobbyDeps` and the graced lock (finding 3).
- D1 and D2, every `Live` prober (`battencli/dobbyreads.go:327`, `battencli/wire.go:713`, `orchcli/cli.go:256`, `loomcli/pawlwiring.go:131,285`, `ensureOne` at `dobbyengine/start.go:227`): survives. Each one tries the lock and releases it.
- D2, state record pid read before the lock against writers of the pid (`recordStart` at `loop.go:191-204`, and `tick` at `loop.go:275` when the pid is 0): survives.
- D4, `State.Tidy` against `tidyStep` / `settleTidy` (`tidy.go:135-215`): survives. A Tidy record is settled whatever the pid, so a second dobby that wins the lock also keeps it.
- D5, `dobby.log` writers:
  - `logLine` (`passes.go:256`) and the stdout `Log` seam (`dobbydeps.go` `Log: fmt.Fprintln(logOut…)`): replaced as decided.
  - `logger` stderr lines that reach `dobby.log` today, the tick-failure episode (`loop.go` `d.failures.Fail`) and the owner-read episode (`beamOwnedElsewhere`): they move to `dobby.out` with no `dobby.log` line taking their place (finding 1).
  - `runDobby`'s pre-`Run` error envelopes (`cli.go:376,385`): move to `dobby.out` (finding 2).
  - Readers of the file: `docs/overview.md:448` and `beamcli/doc.go:117` are doc text only. No Go reader. Survives.
- D5, `SpawnDetached` (`start.go:258-284`) as the one opener of `dobby.out`: survives, given the rotation is at spawn only.
- D7 and D8, Land Done entry:
  - `appendOrFold` / `foldsInto` (`shedengine/run.go:683-713`) folds only budget-exempt Stucks, so a Land Done is never folded: survives.
  - Other `HistoryEntry` writers (`goto.go:135,249`, `entry.go:51`, `reroute.go:45-66`) write no Land Done: survives.
  - `DobbyLandings` (`dobbyreads.go:213`) is the one reader of the count: survives.
- D8 bound, `LandedSinceTidy`: cleared only by `settleTidy` (`tidy.go:203`); `recordStart` and `tick` never touch it. Survives, so the bound's fact holds in code.
- D9, `adopt` key against `MigrateConfig` (`gateslot/config.go`, wired at `configreg.go:128`), `hubreconcile.Ensure` once per build, and `decodeGates`: survives. Until the migration runs, the per-name template merge gives the template value.
- D10, `suite.log` and the envelope:
  - `startSuite` (`adoptcli/deps.go:198`) truncates the log.
  - `result.Suite` and `SuiteWarning` (`adopt.go:325-329`, `cli.go:100-104`) are the envelope's only consumers. Changed as decided: survives.
  - `AdoptScratchDir` is named only by `fabricengine/junctionnames.go:166` and `lymdirs/dirs.go:31`, with no sweep or cleanup reader. Survives: `check.json` persists until the next check.
- D11, `check.json`: no existing reader or writer.

**Findings**

1. LOW, design, Decision 5 and Decision 6.
   Moving stderr to `dobby.out` takes out of `dobby.log` the `logger` episode lines that reach it today: the tick-failure episode, the unreadable-owner episode and the board-tidy strand stop failure.
   Decision 6's list holds only failing *passes*, so a dobby whose ticks fail, for example on an unwritable state record, shows nothing but heartbeats in `dobby.log`.
   The record names the move ("logger lines that reach stderr") but not this loss.
   Evidence: `internal/dobbyengine/loop.go` (`d.failures.Fail("dobbyengine: a tick failed; the loop goes on"…)`, `beamOwnedElsewhere`'s `d.ownerRead.Fail`), `internal/dobbyengine/tidy.go:224` (`stopTidyStrand`'s Warn), and the investigation's "logs at Warn (which reaches dobby.log through stderr)".
2. MEDIUM, design, Decision 3 and Decision 5.
   `runDobby`'s exits before `Run` leave no `dobby.log` line: a `placeDobby` refusal (`cli.go:374-377`) and a `hubDobbyDeps` wiring failure (`cli.go:384-387`).
   After Decision 5 they print only an error envelope into `dobby.out`.
   This is exactly the deploy case: a re-exec'd new binary whose `placeDobby` or wiring refuses the worktree.
   Decision 3's list ("…a fatal error out of `Run`") and the Goal's "never ends silently" do not cover it, and a `placeDobby` failure has no `Geometry.LogPath` to write to.
   Evidence: `internal/beamcli/cli.go:372-392`, `internal/beamcli/dobbydeps.go:69-83` (`ResolveWorktree`, `PauseGeometry`, `noticeProbeInterval` errors).
3. LOW, design, Decision 1.
   With registration after the graced lock, a freshly spawned dobby stays unregistered through `hubDobbyDeps` and up to `lock.ProbeGrace`.
   During that time pair teardown's `battenAlive` reads no live dobby record under the run's parent, and `EndSparing` for a beam slug ends nothing.
   A beam dobby spawned in that window escapes the teardown that should end it.
   The re-exec case is unaffected, since the old image's record keeps the pid.
   The Decision's "what a loser can reach" text says nothing about this widened window for a winner.
   Evidence: `internal/pairteardown/teardown.go:596-629`, `internal/beamcli/dobbydeps.go:57-64` (`registerDobby` owner is the beam slug).

**Tool calls**

- grep for `dobby.log` / `LogPath` readers and `HistoryEntry` writers: the manifest had no reader list for either.
- `appendOrFold` / `foldsInto` and the `HistoryEntry` fields: whether a Land Done can fold and keep an older machine.
- grep for `AdoptScratchDir`, `suiteStart`, `Suite` and the envelope fields: consumers of the adopt envelope and the scratch dir.
- grep for `AdoptDirName` and a read of `owned.go`: whether anything sweeps the adopt dir, and the state writes on the owned-elsewhere path.
- grep of pairteardown's `runproc` and dobby use, and `runproc.End` / `EndSparing`: whether teardown reads the dobby's record.
