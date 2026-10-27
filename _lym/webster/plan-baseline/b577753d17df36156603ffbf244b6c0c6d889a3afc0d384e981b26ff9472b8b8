# Card 6 — dobby-log-file

**Edit:**
- `internal/dobbyengine#Geometry`
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#LogFileName`
- `internal/dobbyengine#daemon.logLine`
- `internal/dobbyengine#SpawnDetached`
- `internal/dobbyengine#TestLog_OneLinePerStartAndPass`
- `internal/hubgeom#DobbyGeometry`
- `internal/hubgeom#TestDobbyGeometry`
- `internal/beamcli#hubDobbyDeps` -> `func hubDobbyDeps(prime *lymcwd.Location, geom dobbyengine.Geometry) (dobbyengine.Deps, error)`
- `internal/beamcli#runDobby`
- `internal/beamcli/doc.go`

**Create:**
- `internal/dobbyengine#OutFileName`
- `internal/dobbyengine#logRotateBytes`
- `internal/dobbyengine#AppendLog`
- `internal/dobbyengine#RotatePastBound`
- `internal/dobbyengine#rotateOutBeforeSpawn`
- `internal/dobbyengine#TestAppendLog`
- `internal/dobbyengine#TestRotateOutBeforeSpawn`
- `internal/dobbyengine#spawnDetached`
- `internal/dobbyengine#TestSpawnDetached_SendsOutputToOutPath`
- `internal/hubgeom#DobbyBeamDir`

**Extends:**
- `internal/dobbyengine/doc.go`

**Uses:**
- `internal/dobbyengine#Live`
- `internal/dobbyengine#newPassHarness`
- `internal/hubgeom#dobbyHub`

**Intent:**
Decision 5: `dobby.log` holds only the dobby's own lines, each appended by a per-line open, rotated to one previous file at 1 MiB, and the detached process's raw output goes to `dobby.out`.

In `dobbyengine`:
- `OutFileName` is the raw-output file beside `LogFileName` in `.lym/beam/`; `LogFileName`'s comment now says it holds the dobby's own lines only. `dobby.log.1` and `dobby.out.1` are named only as `path + ".1"` inside this package, beside their siblings (PATTERN-durable-vs-ephemeral-state).
- `Geometry` gains `OutPath`, the raw-output file under the worktree's `.lym/beam/`, and `PrimeLogPath`, the prime's `dobby.log`, which a beam dobby's worktree-gone line goes to (card 8); both told, never derived (PATTERN-cwd-resolution).
- `AppendLog` opens `path` with append, write-only and create, writes `line` and a newline, and closes it. It never creates a missing directory, so a removed worktree gets nothing back; a missing directory returns the open error. With `rotate` set, a write that would take the file past `logRotateBytes` first renames `path` to `path + ".1"`, replacing an older one, and writes to a fresh file; without it the file grows past the bound. Its doc comment states that only the holder of the file's dobby lock passes `rotate`, so two renames never race.
- `RotatePastBound` renames `path` to `path + ".1"` when the file is at or past `logRotateBytes`, and reports whether it did; an absent file is no rotation and no error.
- `Deps.Log` becomes `func(path, line string, rotate bool)`, its doc no longer listing which events log (the old "one for each task start and each pass spawned" goes, since card 9 changes the events), documented: the dobby passes `Geometry.LogPath` with `rotate` true for its own lines while it holds its lock; a nil Log logs nothing. A struct field takes no re-sign arrow.
- `logLine` passes `d.geom.LogPath` and `true`.
- `SpawnDetached` opens `geom.OutPath` for append, not `LogPath`, and points the child's stdout and stderr there. Before opening, it calls `rotateOutBeforeSpawn(geom, runtime.GOOS)`, which on `windows` alone, and only when `Live(geom)` reads the lock free, calls `RotatePastBound(geom.OutPath)`; a failed rename, as when a refused process still holds the file, is logged through `logger.Warn` and leaves the file for the next spawn. This is the one rotation a process without the dobby lock makes (Decision 5's Windows exception), bounded by the free-lock read.
  `SpawnDetached` keeps its `testing.Testing()` guard and calls `spawnDetached(geom, (*exec.Cmd).Start)`, which holds the rest of the body: the rotation, the open of `OutPath`, the command built with stdout and stderr on that file, and the start through `start`; the spawn's Info log reads the pid only when `cmd.Process` is set, so a fake start that starts nothing logs no pid.

In `hubgeom`: `DobbyBeamDir` returns `<HubPath>/<WorktreeName>/<AnchorRel>/.lym/beam` for a location, the rule `DobbyGeometry` uses; `DobbyGeometry` computes its beam directory through it (from the prime's location for the prime, from the beam's `NamedWorktree` location for a beam), fills `OutPath` beside `LogPath`, and fills `PrimeLogPath` from `DobbyBeamDir(prime)`. Card 10's refusal path shares the function.

In `beamcli`: `hubDobbyDeps` drops its `logOut` writer and wires `Log` to `dobbyengine.AppendLog`, a failed append logged at `logger.Debug`; `runDobby`, its one caller, stops passing its output writer.
`internal/beamcli/doc.go`'s log bullet now says each line is appended to `dobby.log` by its own open and the process's raw output goes to `dobby.out`; `internal/dobbyengine/doc.go` gains the rotation rule and the raw-output file.

Tests:
- `TestAppendLog` (tier0, temp files): an append below the bound adds the line; a rotating append past the bound moves the old content to `.1`, replacing an existing `.1`, and leaves a file holding the one new line; a non-rotating append past the bound grows the file and writes no `.1`; an append into a missing directory returns an error and creates no directory.
- `TestRotateOutBeforeSpawn` (tier0): with `OutPath` past the bound, `windows` and a free lock rotates it; `windows` with the lock held by the test through `lock.TryAcquireWriteLock` leaves it; `linux` with a free lock leaves it; `windows`, a free lock and a file under the bound leaves it.
- `TestSpawnDetached_SendsOutputToOutPath` (tier0, temp `.lym/beam` directory holding a `dobby.log` with one line): `spawnDetached` with a `start` fake that captures the command and starts nothing creates `OutPath`, the captured command's stdout and stderr are both the file opened at `OutPath`, its args are `beam dobby <WorktreeRoot>` in `HubRoot`, and `dobby.log` is byte-identical afterwards; a `start` fake that fails returns its error.
- `TestLog_OneLinePerStartAndPass` records through the new three-argument fake and asserts each line was written to `LogPath` with `rotate` true.
- `TestDobbyGeometry` asserts `OutPath` is `dobby.out` beside `LogPath` and `PrimeLogPath` is the prime's `dobby.log` for each row.

**ImpactSummary:** The Log seam's shape changes for its one production wiring and one test; the spawn's output file moves; two new geometry paths are additive.

**Commit:** 6: Append each dobby log line by itself, rotate at 1 MiB, and send raw output to dobby.out
