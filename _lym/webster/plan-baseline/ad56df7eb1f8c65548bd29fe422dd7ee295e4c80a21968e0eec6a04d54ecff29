# Card 6 — board-serve-open-verbs

**Create:**
- `internal/runproc#RoleBoardServer`
- `internal/boardengine#IsRunAgentSession`
- `internal/boardcli#refuseRunAgent`
- `internal/boardcli#openDeps`
- `internal/boardcli#openResult`
- `internal/boardcli#runOpen`
- `internal/boardcli#spawnServer`
- `internal/boardcli#startOpener`
- `internal/boardcli#openerName`
- `internal/boardcli/open_test.go`
- `internal/boardcli#TestRunOpen`
- `internal/boardcli/serveopen_test.go`
- `internal/boardcli_test#TestServeOpenRefusals`
- `internal/boardcli/serve_linux_integration_test.go`
- `internal/boardcli_test#TestBoardServeScenario`

**Delete:**
- `internal/boardengine#isRunAgentSession`

**Edit:**
- `internal/boardengine#checkTaskCreation`
- `internal/boardengine#Board.UpsertTaskWithRecord`
- `internal/boardcli#Command`
- `internal/runproc#TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast`
- `internal/runproc/doc.go`

**Extends:**
- `internal/boardcli/doc.go`

**Uses:**
- `internal/boardengine/doc.go`
- `internal/boardcli#strandNameFromEnv`
- `internal/boardcli_test#runCLI`
- `internal/boardserve#Paths`
- `internal/boardserve#Live`
- `internal/boardserve#ReadState`
- `internal/boardserve#Probe`
- `internal/boardserve#WriteLauncher`
- `internal/boardserve#PageAddress`
- `internal/boardserve#Run`
- `internal/boardserve#ErrRunning`
- `internal/fabricengine#PrimeName`
- `internal/fabricengine#HubScratchDir`
- `internal/fabricengine#RunProcDir`
- `internal/lymcwd#NamedWorktree`
- `internal/shuttleengines#KeepWarm`
- `internal/selfexec#LymBinary`
- `internal/runproc#SelfRecord`
- `internal/runproc#HubOwner`
- `internal/hubforge#NewHub`
- `internal/hubforge#AddPair`
- `internal/testkit/lymbin#Build`
- `internal/testkit/lymbin#RunWithEnv`

**Intent:**
Add the operator verbs `lym board serve` and `lym board open` (D5, D11, D12) to the board cobra subtree, the opener seam and its per-platform files, and the board server's `runproc` role.

- `runproc` gains `RoleBoardServer`, kept out of `subjectPassSpared`, so a hub takeover by another machine ends the server with the hub's other processes; `doc.go` names it among the hub's roles.
  `TestEndSubject_EndsTheRunsThenTheSubjectAndItsDobbyLast` registers a `RoleBoardServer` record under `HubOwner` beside the spared ones and asserts `EndSubject` ends it.
- `boardengine.IsRunAgentSession` replaces the unexported `isRunAgentSession` with the same rule (a strand name with a task slug under any role but the orch's); its two callers, `checkTaskCreation` and `Board.UpsertTaskWithRecord`, call it.
- `refuseRunAgent(verb, strand)` refuses for a run agent's strand name:
  `the run session <strand> cannot run lym board <verb>: the board viewer and its server are the operator's tools, and a server an agent started would carry its session into the operator's page: way forward: ask your orch or the operator to run lym board open`.
  An orch's strand name and an empty one pass.
- Both verbs live in `Command` beside the other verbs. Each RunE first runs `refuseRunAgent` with `strandNameFromEnv()`, then refuses `--board-path` (the PersistentPreRunE leaves `location` nil):
  `lym board <verb> needs the hub's board config, which only a run from a worktree of the hub resolves: way forward: run it from a worktree of the hub, without --board-path`.
  Neither refusal starts a process.
  `open` is audience operator and takes no argument; `serve` is audience internal, since `open` spawns it.
- `rerender`'s `Short`, which names the README and design docs alone, becomes "rebuild the board's rendered files from board.json", since a render now also writes `board-data.js` and `board.html`.
- `serve` builds its board once, `boardengine.New` over the PersistentPreRunE's config with `StrandName` emptied, so every page write is the operator's; the config already carries the board's skip variables through `ApplySkipEnv`.
  It runs `boardserve.Run` with `Paths(fabricengine.HubScratchDir(hub))`, `Register` writing `runproc.SelfRecord(RoleBoardServer)` under `HubOwner` in `fabricengine.RunProcDir(hub)`, `Replaced` from `selfexec.New()`, `fswatch.Watch`, the wall clock and stderr as the log.
  `ErrRunning` prints `{"ok":true,"running":true}` and exits 0, as creel's daemon does; the opener's wait then finds the winner.
- `open` builds `openDeps` from production values and prints `runOpen`'s result as `{"ok":true,"address":<PageAddress>,"launcher":<path>,"started":<bool>}`; the address carries no token.
  `openDeps` holds `Paths`, `Prime` (resolves the prime's anchored directory), `Spawn`, `Probe`, `Running`, `StartOpener`, `StartBound`, `Poll` and `OpenerCap`.
  `OpenerCap` zero means no cap: the wait lasts until the opener exits.
  The verb sets it to `shuttleengines.KeepWarm(fabricengine.ConfigDirs(location), strand).Cap` when `strandNameFromEnv()` is not empty, and leaves it zero for the operator's call with no strand name.
  `runOpen` does:
  1. `refuseRunAgent` is already passed; `Live(paths)` decides the path.
  2. Lock held: wait, bounded by `deps.StartBound` (production: 15 seconds, re-reading every `deps.Poll`, production one second), re-reading the state on each try, until a state whose pid `deps.Running` reports running (production: `proc.IsRunning`) answers `deps.Probe` with its token.
  3. Lock free: `deps.Prime()` resolves the prime's anchored directory with `fabricengine.PrimeName` and `lymcwd.NamedWorktree(hub, prime).AnchorPath()`, whatever worktree `open` ran from; its error is returned as D5 states and nothing starts.
     `deps.Spawn(primeAnchor, paths.Log)` starts the server, then waits as in 2 until the lock is held and the new state answers.
     A spawn error is returned: `start the board server: <err>: way forward: check that lym runs from this shell and read the server log <path>, then run lym board open again`.
  4. Either wait ending at the bound refuses: `the board server did not answer within <bound>; its log <path> opens with its pid: way forward: end that process if it hangs, then run lym board open again`.
     Card 5's `Run` truncates the log and writes its pid first once it holds the lock, so the log names the server that holds it.
  5. `WriteLauncher` writes the launcher; `deps.StartOpener(launcher)` starts the opener.
     A start or exit error returns `the opener <openerName> failed: <err>: way forward: open <launcher path> in a browser by hand`.
     With a non-zero `OpenerCap`, the wait on the opener ends at the cap; the verb leaves the opener running and returns success.
     With a zero cap it waits until the opener exits.
- `spawnServer` starts `<selfexec.LymBinary()> board serve` with `Dir` the prime's anchor, stderr appended to the server log, `proc.Detach`, an `Info` spawn log through `internal/logger`, and `proc.Reap`.
- `startOpener` lives in `opener_linux.go` (`xdg-open <path>`) and `opener_windows.go` (`rundll32 url.dll,FileProtocolHandler <path>`), each declaring `openerName`.
  It logs the spawn, detaches it with `proc.Detach`, and waits on it in one goroutine whose `cmd.Wait` result feeds the returned channel; that goroutine is the opener's reap.
- `doc.go` gains a paragraph on `open` and `serve`.

Bound: `open` reaches only the hub's own server and the default browser; `serve` writes the board through `boardengine` alone (card 4's bound).
A session that clears its strand-name variable passes the run-agent refusal, as it passes `lym board upsert`'s task-creation rule.

`TestRunOpen`, tier0 in `internal/boardcli/open_test.go` (package `boardcli`), drives `runOpen` with fake `Prime`, `Spawn`, `Probe`, `Running`, `StartOpener` and short bounds over real lock files in a temp state directory:
- the lock held by a live server whose state answers: no spawn, and the opener gets a launcher path under the state directory whose argument holds no token;
- the lock held with a previous server's state left, and the new server's state written mid-wait: the result names the new server's address;
- the lock held and no state ever answering, or a state whose pid is not running: refused naming the server log, and no spawn;
- the lock free with the previous server's state still on disk naming a port, as after an exit or during a drain: one spawn in the prime's anchored directory the fake `Prime` gives, a wait for the new state, then the launcher; a server that never answers is refused naming the log;
- the lock free and `Prime` failing: its error is returned and nothing spawns;
- the lock free and `Spawn` failing: the error names the server log and its way forward, and no opener starts;
- the address holds no token;
- an opener that fails returns an error naming `openerName` and the launcher path to open by hand;
- a non-zero `OpenerCap` and an opener that never exits: success at the cap;
- a zero `OpenerCap` and an opener that exits after the test releases it: `runOpen` returns only after that exit.
`TestServeOpenRefusals`, tier0 in `internal/boardcli/serveopen_test.go` (package `boardcli_test`), not parallel since it sets `LYM_STRAND_NAME`, runs both verbs through `runCLI` with `--board-path` set to a temp directory, since a tier0 test makes no git hub for the cwd resolution:
with `LYM_STRAND_NAME` set to `""` through `t.Setenv`, each refuses naming a run from a worktree of the hub;
with a run agent's strand name, each refuses naming the operator's tool, the run-agent check coming before the `--board-path` refusal;
no process starts.
`TestBoardServeScenario`, tier1 in `internal/boardcli/serve_linux_integration_test.go`, runs a built `lym` (`lymbin.Build`) in a `hubforge` hub with a task pair, `BOARD_SKIP_GIT=1`, `LYM_STRAND_NAME=` in every call's extra environment, and a stub `xdg-open` first on `PATH` that records its argument:
`lym board open` from the pair's worktree answers `started: true`, its address holds no token, the stub's argument is the launcher path, and the server's `/proc/<pid>/cwd` is the prime's anchored directory;
a second `open` answers `started: false`;
`lym board serve` run from the prime's anchor with the same extra environment exits 0 with `running: true` in its envelope, while the first server's state still answers `Probe`;
cleanup ends the server by its state's pid.

**ImpactSummary:** boardcli gains two verbs inside Command and rewords rerender's Short; boardengine's run-agent rule becomes exported with its two in-package callers; runproc gains one role constant outside subjectPassSpared, so EndSubject ends it.

**Commit:** 6: board: lym board open and lym board serve
