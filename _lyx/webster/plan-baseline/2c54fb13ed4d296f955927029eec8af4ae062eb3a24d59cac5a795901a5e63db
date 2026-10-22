# Card 3 — Detached pushes and board syncs register under the owner D2 names

**Create:**
- `plan:internal/lyxdirs#RunProcDirName` -> `const RunProcDirName = "runproc"`
- `plan:internal/fabricengine#RunProcDir` -> `func RunProcDir(hub string) string`
- `plan:internal/fabricengine#startDetachedPush` -> `func startDetachedPush(exe string, req PushSpawn) error`
- `plan:internal/fabricengine#TestStartDetachedPush_RegistersTheChild` -> `func TestStartDetachedPush_RegistersTheChild(t *testing.T)`
- `plan:internal/boardengine#TestSpawnSync_RegistersUnderTheHubOwner` -> `func TestSpawnSync_RegistersUnderTheHubOwner(t *testing.T)`

**Edit:**
- `internal/fabricengine#SpawnDetachedPush`
- `internal/fabricengine#pushSpawnCommand` -> `func pushSpawnCommand(req PushSpawn) (args []string, dir, owner string)`
- `internal/fabricengine#TestPushSpawnCommand`
- `internal/boardengine#spawnSync`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/runproc#Register`
- `internal/runproc#ChildRecord`
- `internal/runproc#List`
- `internal/runproc#HubOwner`
- `internal/runproc#RolePush`
- `internal/runproc#RoleSync`
- `internal/fabricengine#HubScratchDir`

**Intent:**
Declare the owner record's directory and register the two detached spawners of D2 right after `Start`, so every respawn path (commit, pair push, pending-flag respawn, teardown push) is covered by the one start site each has.

Directory (`PATTERN-told-geometry`, `PATTERN-lyxdirs-single-declarer`).
`lyxdirs.RunProcDirName` is `runproc`, declared beside `FailedPushDirName`.
`fabricengine.RunProcDir(hub)` returns `<HubScratchDir(hub)>/runproc`, declared in `junctionnames.go` beside `LogArchiveDir`, the one constructor every caller hands to `runproc`.

Push (D2).
`pushSpawnCommand` also returns the owner: `req.TeardownSlug` when set (a task pair's push and every respawn of one), and `runproc.HubOwner` otherwise (a prime records push, whatever `PauseSlug` it carries), so a pair's teardown never ends a prime records push.
`SpawnDetachedPush` keeps its skip-env, empty-path and test-binary early returns, then calls `startDetachedPush(exe, req)`, which builds the command, `proc.Detach`es it, starts it, logs the spawn as today and then calls `runproc.Register(RunProcDir(filepath.Dir(dir)), owner, runproc.ChildRecord(pid, runproc.RolePush, args))`, where `dir` is the request's working directory, always `BoardDir(hub)`.
A registration failure is logged at Warn and never fails the spawn, since the push itself is already running.
`TestPushSpawnCommand` gains the expected owner on each existing row: the task-pair rows own under `task`, the prime rows under `runproc.HubOwner`.
`TestStartDetachedPush_RegistersTheChild` (Linux only, skipped elsewhere) calls `startDetachedPush` with the `true` binary from `exec.LookPath` as `exe`, a pair request whose `WorkDir` is a temp hub's board dir, and asserts one record file under `RunProcDir(hub)/task` with role `push` and the child's pid; a prime request lands under `HubOwner`;
a request whose `RunProcDir` path is taken by a plain file still starts the child and returns nil, logging the registration failure at Warn (`internal/testkit/logcapture`).
Each row reaps the child it started with `syscall.Wait4` on the recorded pid in `t.Cleanup`, so no process outlives the test.

Board sync (D2).
`spawnSync` registers its started child under `runproc.HubOwner` with role `sync` in `fabricengine.RunProcDir(filepath.Dir(abs))`, after its existing spawn log, with the same Warn-only failure.
Its executable comes from a package var defaulting to `os.Executable`, so `TestSpawnSync_RegistersUnderTheHubOwner` (Linux only) points it at `true`, calls `spawnSync` on a temp board dir and asserts the record file under `HubOwner`, reaping the child in `t.Cleanup` the same way; a board dir whose `RunProcDir` path is taken by a plain file still spawns and returns nil, logging one Warn.

The `fabricengine` doc's pending-flag and teardown-lock section names the registration, its owner rule and `RunProcDir`.

**ImpactSummary:** pushSpawnCommand gains a result, its only callers are SpawnDetachedPush and TestPushSpawnCommand; the two spawn sites gain a Warn-only registration.

**Commit:** 3: detached pushes and board syncs register in the owner record
