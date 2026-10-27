# Card 10 — dobby-cli-exits-and-out

**Edit:**
- `internal/beamcli#runDobby`
- `internal/beamcli#placeDobby` -> `func placeDobby(root string) (*lymcwd.Location, *lymcwd.Location, dobbyengine.Geometry, error)`
- `internal/beamcli/doc.go`

**Create:**
- `plan:internal/beamcli#runDobbyWith` -> `func runDobbyWith(ctx context.Context, out io.Writer, root string, wire func(prime *lymcwd.Location, geom dobbyengine.Geometry) (dobbyengine.Deps, error), notify func(from *lymcwd.Location, slug, program, line string) (bool, error)) int`
- `plan:internal/beamcli#reopenStdio` -> `func reopenStdio(path string) error`
- `plan:internal/beamcli#TestRunDobbyWith_TracesEachExitBeforeRun` -> `func TestRunDobbyWith_TracesEachExitBeforeRun(t *testing.T)`
- `plan:internal/beamcli_test#TestDobby_ReexecsOntoAReplacedBinary` -> `func TestDobby_ReexecsOntoAReplacedBinary(t *testing.T)`

**Extends:**
- `internal/dobbyengine/doc.go`

**Uses:**
- `internal/beamcli#hubDobbyDeps`
- `internal/beamcli#registerDobby`
- `plan:internal/dobbyengine#AppendLog`
- `plan:internal/dobbyengine#RotatePastBound`
- `plan:internal/dobbyengine#ReadsAsReexec`
- `internal/dobbyengine#Run`
- `internal/dobbyengine#Live`
- `internal/dobbyengine#State`
- `plan:internal/hubgeom#DobbyBeamDir`
- `internal/beamcli#seedBattenRun`

**Intent:**
Decisions 3 and 5 at the CLI: each exit `runDobby` takes before `Run` leaves its trace, and a dobby moves its raw output off `dobby.log` onto `dobby.out` at its first start on this binary.

`runDobby` becomes `runDobbyWith(ctx, out, root, hubDobbyDeps, orchcli.NotifyFromScoped)`; the seams let a test drive a failing wiring and record the notices.

`placeDobby` returns the location `lymcwd.ResolveWorktree` resolved first, before the prime and the geometry, and keeps it non-nil on every refusal made after that resolve succeeded (`RequireDrivableWorktree`, `PrimeName`, `ReadOrigin`, the neither-prime-nor-beam check, the prime's resolve, `hubgeom.DobbyGeometry`); a refusal at `filepath.Abs` or `ResolveWorktree` returns a nil location.

`runDobbyWith`, in order:
1. A `placeDobby` refusal with a resolved location appends `<time> start refused: <error>` to `dobby.log` in `hubgeom.DobbyBeamDir(location)` through `dobbyengine.AppendLog` with `rotate` false; a worktree with no `.lym/beam` directory, such as a pair that was never a beam, gets no line, since the append creates no directory. It runs `dobbyengine.ReadsAsReexec` on that directory's `state.json` with this pid and `proc.StartTime`, and on a re-exec posts `<worktree>: the dobby could not start after a re-exec: <error>; way forward: fix the cause, then run lym beam start` through `notify` over the location, signed `DOBBY` and scoped to the worktree's name unless it is the prime; the refusal's own `lym beam start` alone would only repeat the refusal. A refusal with no location keeps only its envelope, which goes to `dobby.out` for a process this binary spawned (card 6). It returns the error envelope as now.
2. Where `selfexec` execs, that is off Windows and outside a test binary, it reopens its own stdout and stderr onto `geom.OutPath` for append through `reopenStdio`, before anything else writes, so a dobby whose fds an older binary pointed at `dobby.log`, living on by re-exec, moves them off it. `reopenStdio` opens the file and duplicates it onto fds 1 and 2 with `unix.Dup2` in a `!windows` file, and is a no-op returning nil in a `windows` file. A failed reopen is logged through `logger.Warn` and the dobby goes on.
3. A `wire` failure appends `<time> start failed: wire the dobby over the hub: <error>` to `geom.LogPath` with `rotate` false; when `ReadsAsReexec` passes on `geom.StatePath` it resolves `geom.WorktreeRoot` through `lymcwd.ResolveWorktree` itself and posts `<worktree>: the dobby's wiring failed after a re-exec: <error>; way forward: fix the cause, then run lym beam start` through `notify`; when that resolution fails the line is the only trace. It returns the error envelope as now.
4. It sets `deps.Locked` to one callback, which `Run` calls only once it holds the lock: it registers the dobby (`registerDobby`, as card 7 set it) and then, where `selfexec` execs, runs `dobbyengine.RotatePastBound(geom.OutPath)` and, when it rotated, `reopenStdio(geom.OutPath)` onto the fresh file; so only the lock holder rotates `dobby.out`, and a process that loses the lock renames nothing.
5. It runs `dobbyengine.Run` and prints its outcome as now.
Bound: a wiring failure on a worktree whose `.lym/beam` directory does not exist yet loses its line, since no append creates a directory; a refused start writes its envelope to the file it holds, which a winner's rotation may have renamed to `dobby.out.1`.

Tests:
- `TestRunDobbyWith_TracesEachExitBeforeRun` (tier1, over a `hubforge` hub with a beam pair and a task pair, notices recorded by the `notify` fake, rows in sequence over one hub): a failing `wire` on the prime with a `.lym/beam` directory and no state record writes the start-failed line to the prime's `dobby.log` and posts nothing; the same with a state record holding this test's pid and no ticks also posts one notice naming the wiring error and `fix the cause, then run lym beam start`; the same with a `wire` fake that removes the prime's worktree link, so the resolve of `geom.WorktreeRoot` fails, before it fails writes its line and posts nothing; the task pair with a `.lym/beam/state.json` holding this pid, refused as neither prime nor beam, gets the start-refused line in its own `dobby.log` and one notice naming the refusal's error and `fix the cause, then run lym beam start`; the task pair without a `.lym/beam` directory gets no line, no notice and no directory; a loose directory of the hub gets no notice and no `dobby.log`, its envelope on the call's output; that this envelope lands in `dobby.out` for a process this binary spawned follows from the spawn pointing stdout there, which card 6's `TestSpawnDetached_SendsOutputToOutPath` pins.
- `TestDobby_ReexecsOntoAReplacedBinary` (tier1, `beamcli_test`, the acceptance test of Decisions 1 to 5): it skips on Windows, where `selfexec` never execs; it copies the prebuilt `lym` from `lymbin.Build` into a temp directory, seeds the prime's `dobby.out` past 1 MiB, starts `<copy> beam dobby <prime root>` detached from the hub root with its stdout and stderr on the prime's `dobby.log`, as an older spawner left them, registers a `t.Cleanup` that kills the process if it is still alive, so a failing run leaves no dobby behind, and waits for `dobbyengine.Live`. It asserts `dobby.out.1` holds the seeded content and `dobby.out` is under the bound, the lock holder's rotation at its start. It writes a `Tidy` whose `Deadline` is a day ahead into the state record through `dobbyengine.UpdateState`, so the tidy step neither settles nor times it out, starts a goroutine looping on `dobbyengine.Live`, replaces the binary by writing a fresh copy and renaming it over the path, and writes a file into the prime's batten runs directory once a second, so the watch wakes the loop, until `dobby.log` holds the re-exec line and a `start re-exec` line, bounded at three minutes, since where the watch does not open the loop ticks at most once a minute and the re-exec needs two sightings. It then asserts the state record still holds the process's pid and the `Tidy`, `Live` reads true, sends SIGTERM, and asserts `dobby.out` holds the stopped envelope while `dobby.log` holds no JSON envelope. It is tier1 because it needs a real exec, which `selfexec` refuses under `go test`, and tier1 sets `prebuilt_lym` (PATTERN-test-speed).

Docs: `internal/beamcli/doc.go`'s dobby section gains the traces before `Run`, the stdout reopen and the rotation of `dobby.out`; `internal/dobbyengine/doc.go` gains the `dobby.out` bound, that it grows by stderr alone between starts of the lock holder.

**ImpactSummary:** `runDobby` keeps its signature and envelope; `placeDobby`'s re-sign reaches only its one caller, now `runDobbyWith`.

**Commit:** 10: Trace each dobby exit before Run and move its raw output onto dobby.out
