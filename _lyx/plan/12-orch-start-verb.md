# Card 12 — orch-start-verb

**Create:**
- `internal/orchcli/start.go`
- `internal/orchcli/spawn.go`
- `internal/orchcli/handover.go`
- `internal/orchcli/start_test.go`
- `internal/orchcli/handover_test.go`

**Edit:**
- `internal/orchcli/cli.go`
- `internal/orchcli/prime_integration_test.go`
- `cmd/lyx/helptree_test.go`

**Uses:**
- `internal/loomcli/start.go`
- `internal/loomcli/driverlaunch.go`
- `internal/orchengine/launch.go`
- `internal/orchengine/state.go`
- `internal/orchengine/watchloop.go`
- `internal/orchengine/prompt.go`

**Intent:**
Add `lyx orch start [--no-attach] [--handoff <path>]`, the idempotent bootstrap of D3, D5 and D6.

The receiver gains two seams beside `strandOps`, so untagged tests exercise every branch without a spawn:
`sessionStarter` (`StartSession(spec shuttleengine.Spec) (guid string, err error)`, adapting `Runner.Start` the way `loomcli`'s `runnerDriverStarter` does) and `spawnWatcher func() error`.

`start.go`'s `RunE`, every fallible step reporting on the envelope before any handover:

1. Create `Paths.Dir` and hold `StartLockPath` (`lock.AcquireWriteLock`) until the handover, so two concurrent starts cannot both relaunch.
2. `reed.Up()`.
3. Read the strands and pick those named `orch` (see the `orch-strand-identity` decision).
   More than one refuses, naming them.
   Read the watcher's liveness through `orchengine.WatcherLive`.
4. `orchengine.DecideStart(strandLive, watcherLive)`.
   `--handoff` with a live strand refuses: D6 governs a fresh launch only, and a live session must be stopped first.
5. `StartRelaunch`: remove a corpse strand first, so two strands never share the name (D5).
   Then `orchengine.ChooseStartPrompt`, and `StartSession` with the `orch-session-spec` decision's spec.
   Finally save `orchengine.ResetForFreshLaunch(state, guid)`, which resets any persisted phase (D10).
6. A live `orch` strand whose guid differs from `State.Strand` is adopted through the same `ResetForFreshLaunch`, since the persisted phase and offsets belong to another run.
7. `StartRelaunch` and `StartSpawnWatcher` call `spawnWatcher`.
8. Release the start lock and hand over.

`spawn.go`'s production `spawnWatcher` resolves `os.Executable()` and builds `exec.Command(exe, "orch", "watch")` with `Dir` set to the prime's anchor path, so the child's `lyxcwd` resolution finds the prime.
Stdout and stderr go to `WatchLogPath`, opened for append.
It applies `proc.Detach`, logs the spawn at `Info` (pid and log path), and starts the command without waiting, reaping it in a goroutine the way `loomcli`'s go-driver spawn does.
It is a no-op under `testing.Testing()`, per the "never re-exec `os.Executable()` under go test" clause.

`handover.go` decides the terminal handover the way `loom start` does, as a pure `decideHandover(noAttach bool, tmuxEnv string, reedOwns bool, currentSession, targetSession string)`:

- `--no-attach` → envelope.
- `$TMUX` unset → attach through `reed.AttachArgv`, with the terminal size from `term.GetSize`.
- `$TMUX` naming reed's server with this client already in the prime's session → envelope.
- `$TMUX` naming reed's server, another session → `switch-client`.
- `$TMUX` naming a foreign server → envelope with a hint to run `lyx reed attach` from outside tmux.

The attach and switch spawns log before and after, per Live-Substrate Spawn Observability.
They are this command's interactive-handoff exception to the CLI/Cobra Invariant, recorded in `CONSTRAINTS.md` by card 13.
The envelope carries `action` (attach-only, spawned-watcher or relaunched), `strand`, `prompt_source` and `attached`.
The code is duplicated from `loomcli` rather than imported, for the same cli-to-cli coupling reason as card 10's refusal.

Register `start` in `cli.go`, add it to `helptree_test.go`'s `orch` row, and add it to `prime_integration_test.go`'s verbs so the prime-only refusal is asserted for `start` too.

`start_test.go` drives `RunE` with `--no-attach` over fakes and asserts the recorded calls and the saved state:

- no strand → one `StartSession` with the start stencil, the watcher spawned, state reset onto the new guid
- a live strand and a live watcher → nothing started or spawned
- a live strand and no watcher → the watcher spawned only
- a dead strand → the corpse removed before `StartSession`
- a dead strand with a recorded last handoff → the resume prompt naming it
- `--handoff` → the resume prompt naming the flag's file
- `--handoff` with a live strand → refusal
- two `orch` strands → refusal
- a fresh launch over a persisted non-idle phase → the phase reset to `idle`, the abandoned phase recorded, and nothing sent but the launch prompt
- the spec carries `Interactive`, `AwaitOperator`, the `orch` name, `Focus` and the configured model and effort

`handover_test.go` covers each `decideHandover` row.

**ImpactSummary:** Adds one subcommand to `orchcli`'s tree; outside the module only `cmd/lyx/helptree_test.go` changes, gaining `start` in the `orch` row.
