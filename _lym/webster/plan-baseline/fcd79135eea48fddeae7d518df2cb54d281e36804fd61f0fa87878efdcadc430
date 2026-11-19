# Card 5 — board-server-lifecycle

**Create:**
- `internal/lymdirs#BoardServeDirName`
- `internal/boardserve#StatePaths`
- `internal/boardserve#Paths`
- `internal/boardserve#State`
- `internal/boardserve#ReadState`
- `internal/boardserve#Live`
- `internal/boardserve#PageAddress`
- `internal/boardserve#WriteLauncher`
- `internal/boardserve#Probe`
- `internal/boardserve#Options`
- `internal/boardserve#ErrRunning`
- `internal/boardserve#Run`
- `internal/boardserve#watchBoard`
- `internal/boardserve#hardenFile`
- `internal/boardserve/run_test.go`
- `internal/boardserve#TestRun`
- `internal/boardserve/owneronly_windows_test.go`
- `internal/boardserve#TestHardenFileGrantsOnlyTheOwner`

**Extends:**
- `internal/boardserve/doc.go`
- `cmd/lym#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `internal/lymdirs/doc.go`
- `internal/boardserve#Server.BoardChanged`
- `internal/boardserve#Server.beginDrain`
- `internal/boardserve#Server.lastAPIRequest`
- `internal/boardengine#BoardRevision`
- `internal/lock#AcquireWriteLockWithin`
- `internal/lock#TryAcquireWriteLock`
- `internal/state#WriteJSON`
- `internal/state#ReadJSON`
- `internal/fswatch#Watch`

**Intent:**
Give the board server its single-instance lifecycle, state file, launcher and watch loop (D9, D11).

- `BoardServeDirName` names the state directory under the hub's board-level `.lym` (`fabricengine.HubScratchDir`), beside creel's; that `.lym` is never staged by the board sync, so the token never reaches a commit.
  `Paths(hubScratchDir)` gives the directory and, by named constants, its state file `state.json`, the lock `server.lock`, the launcher `launcher.html` and the log `server.log`.
  The lock is taken only through `internal/lock`'s functions.
- `State` is `{"pid","port","token"}`, written through `state.WriteJSON` (a temp file created 0600, then renamed) and read through `state.ReadJSON`, which ignores an unknown key.
  `hardenFile` sets mode 0600 on every platform; its Windows file (`owneronly_windows.go`, build-tagged) also sets an owner-only protected DACL, as `internal/githubclient/cache_windows.go` does, since Windows ignores Go's permission bits; the other platforms' file is `owneronly_other.go`.
  The state file and the launcher both pass through it.
- `Live` tries the lock without blocking and releases it at once, as `dobbyengine.Live` reads a dobby's: a held lock is a live or starting server, a free one none, whatever a state file says.
  A missing state directory reads as no server, with no error and nothing created, as `dobbyengine.Live` reads a missing lock directory.
- `PageAddress` is `http://127.0.0.1:<port>/`, never `localhost`.
  `WriteLauncher` writes `launcher.html` in the state directory: a small page whose inline script replaces its location with `PageAddress(port)` + `#` + the token; it returns the launcher's path.
  `Probe` sends `GET /api/board` with the `127.0.0.1:<port>` host and the bearer token and succeeds on a 200.
- `Options` holds `Paths`, `Board`, `BoardPath`, `LockWait`, `IdleBound`, `PollBound`, `Log` (the writer the server logs to), `Register`, `Replaced`, `Watch`, `Now` and `After`.
- `Run(ctx, opts)`, the `serve` verb's body:
  1. creates the state directory and takes the lock with `AcquireWriteLockWithin(lock, opts.LockWait)` (production: five seconds), so `open`'s instant liveness read never turns it away; when it is still held, it returns `ErrRunning`, which the verb answers with success, having written nothing to the log or the state;
  2. truncates the server log to empty and writes `lym board serve pid <pid>` as its first line through `opts.Log`, so the log of the server that holds the lock opens with its pid, and removes any leftover state file;
  3. calls `opts.Register` (the verb's `runproc` registration);
  4. binds `127.0.0.1:0`, draws a 128-bit token from `crypto/rand` as 32 hex characters, builds the card-4 `Server`, writes the state through `hardenFile`, and serves;
  5. runs `watchBoard` until it returns an exit reason, or `ctx` ends;
  6. on exit: calls `http.Server.SetKeepAlivesEnabled(false)` and closes the listener, so the server takes no new connection and each kept-alive connection closes after its current response;
     a connection accepted just before may still carry one request, which the board's write lock keeps consistent;
     releases the lock (the state file stays for the next server to remove) and calls `beginDrain`;
     then waits for the requests in flight through `http.Server.Shutdown`, ignoring the error it reports for the already-closed listener, so a write inside the board's write section completes and spawns its sync;
     it returns nil.
- `watchBoard` is the one watch loop: it wakes on an `internal/fswatch` event on `board.json` in the board directory, or on its timer, which starts at one second and doubles to eight while the board's revision is unchanged and returns to one second on a change.
  When `opts.Watch` fails, the timer alone reads `board.json`.
  Each wake recomputes `BoardRevision`; a change calls `BoardChanged`, so a Bolt replay or a sync pull that rewrites `board.json` reaches the page like a local write.
  It returns "replaced" when `opts.Replaced` reports the binary replaced (production: `selfexec.New().Replaced`), and "idle" when `lastAPIRequest` (or the start, before any) is `opts.IdleBound` (production: 30 minutes) behind `opts.Now`.
  The clock is `opts.After` and `opts.Now`; production passes `time.After` and `time.Now`, and a test injects its own.
- `cmd/lym/daemonwakeups_test.go`'s `unboundedWaitFiles` lists `internal/boardserve/run.go`, and `pattern/PATTERN-daemon-wakeups.md`'s list gains the board server's watch loop: an `internal/fswatch` event on `board.json` or a timer from one second doubling to eight, with the binary and idle checks on each wake.
  The long poll of card 4 is bounded under a minute and stays outside the rule.
- `doc.go` gains the lifecycle paragraph.

Bound: `ErrRunning` lets a second server exit without serving, so at most one server holds a hub's lock; the idle and binary exits bound the server's life with no stop verb.

`TestRun`, tier0 in `internal/boardserve/run_test.go` (package `boardserve`), runs `Run` in process on a loopback port over a temp board with an injected clock, `Replaced`, `Watch` and a recording `Register`:
- `Live` over a state directory that does not exist is false with no error and creates nothing;
- with the lock held by the test, `Run` returns `ErrRunning` after its (short, injected) wait and writes no state and nothing to the log;
- over a leftover state file holding an unknown key, `ReadState` reads its pid, port and token; `Run` then removes it, and the new state names the test process's pid, a 32-hex-character token and a port that `Probe` answers with the token;
  over a leftover log, the log's first line names the test process's pid; `Register` was called once;
  off Windows the state file and the launcher are mode 0600; the launcher's address uses `127.0.0.1` and carries the token only after `#`;
- `Live` is true while `Run` serves and false after it returns;
- with no `/api` request, advancing the clock past the idle bound ends `Run` with nil, and the lock is free;
  an `/api` request made before the bound passes, followed by a clock advance to the bound measured from the start, keeps `Run` serving until the bound passes from that request;
- `Replaced` reporting true ends `Run`;
- a write in flight completes: the test holds the board's write lock (`fabricengine.BoltLockPath(boardDir, fabricengine.BoardWriteLockFile)`) so a `POST /api/write` blocks inside the write section, and triggers the idle exit;
  while the write is still blocked, `Live` reads false, the drain having released the server's lock;
  the test then releases the board lock and asserts the write answered 200, `board.json` holds it, and `Run` returned only after;
- a waiting `GET /api/board?after=` answers "unchanged" at once on exit;
- with a working injected `Watch`, a change to `board.json` followed by an event on its channel wakes a waiting `GET /api/board` with the new revision, the injected timer never firing;
- with `Watch` failing, a change to `board.json` still wakes a waiting `GET /api/board` once the injected timer fires.
`TestHardenFileGrantsOnlyTheOwner`, tier0 in `internal/boardserve/owneronly_windows_test.go`, which only Windows compiles, asserts the hardened file's DACL holds one entry, for the current user.

**ImpactSummary:** new package code plus one lymdirs constant; the daemon-wakeups guard and its background list each gain one line, and no existing caller changes.

**Commit:** 5: boardserve: single-instance lifecycle, state, launcher and watch loop
