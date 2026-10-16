# Card 9 — tender-status

**Edit:**
- `internal/pauseengine#Pair`
- `internal/pauseengine#Deps`
- `internal/pauseengine#StatusReport`
- `internal/pauseengine#Status`
- `internal/pauseengine/doc.go`
- `internal/pauseengine#fakePair`
- `internal/pauseengine#newPair`
- `internal/pauseengine#env`
- `internal/pauseengine#env.deps`
- `internal/pauseengine#TestStatus`
- `internal/pausecli#hubDeps`
- `internal/pausecli#TestPauseAndResume_WriteLiftAndRefuseOverTheHub`
- `docs/overview.md`

**Rename:**
- `internal/pausecli#shuttlePair.state` -> `plan:internal/pausecli#shuttlePair.State`

**Create:**
- `plan:internal/pauseengine#SessionStatus` -> `type SessionStatus struct`
- `plan:internal/pauseengine#StaleReport` -> `type StaleReport struct`
- `plan:internal/pauseengine#StaleProcess` -> `type StaleProcess struct`
- `plan:internal/pauseengine#sessionStatuses` -> `func sessionStatuses(deps Deps, record pauserecord.Record) ([]SessionStatus, error)`
- `plan:internal/pauseengine#staleProcesses` -> `func staleProcesses(processes []proc.Process, hubPath, executable string) []StaleProcess`
- `plan:internal/pauseengine#fakePair.State` -> `func (p *fakePair) State(guid string) (shuttleengine.SessionState, error)`

**Intent:**
`lyx tender status` becomes the one read-only view of what runs, waits, stalls or runs stale in the hub (D3, D7).

`Pair` gains `State(guid string) (shuttleengine.SessionState, error)`, the session's state read from its run's files.
`shuttlePair.state` is renamed to the exported `State` to satisfy it; its callers `Busy` and `Background` follow the rename.

`Deps` gains:
- `Prime Pair`, the prime's sessions, nil when the hub has no prime that wires; `Pairs` stays the task pairs only, so pause and resume are unchanged;
- `Processes func() ([]proc.Process, error)`, the process reading of card 7;
- `HubPath string` and `Executable string`, the hub directory and the running lyx's own executable path.

`StatusReport` gains `Sessions []SessionStatus` and `Stale StaleReport`, both always non-nil in the JSON, so each part reads empty when there is nothing to show.
`SessionStatus` carries `Slug`, `Strand`, `State`, `Cause`, `Since`, `Prime bool` and `Paused bool`.

`Status` adds two parts:
- `sessionStatuses` lists every live session of every task pair and then of `Prime`: each pair's `Sessions()` and each session's `State`.
  `Paused` is true when a live entry of the record covers the session's slug.
  A prime session is covered only by a hub entry, since no slug entry ever names the prime, so the orch's session and any other live agent session in the prime are listed with that flag.
  A session whose state reading fails is listed with the state `unknown` and the error as its cause, and the rest of the report is unaffected.
- `staleProcesses` keeps each process whose `Exe` ends in ` (deleted)` and whose base name, with that suffix stripped, equals `Executable`'s base name.
  That holds wherever the binary lives, the Go bin dir and `.dev-bin` alike.
  The process's `Cwd`, with the suffix stripped, must be `HubPath` or lie inside it by path-segment containment, a deleted working directory included.
  Each kept process shows `PID`, `StartTime` and `Argv`; the listing attributes no process to a pair.
  A `Processes` error wrapping `errors.ErrUnsupported` reports `Stale.Supported` false with no processes.
  Any other error is reported in `Stale.Error`, and in both cases the rest of the report is unaffected.

`Status` reads no transcript and no pane content and writes nothing, so any caller may run it (D3).
`pauseengine/doc.go`'s "Status" section states the two parts.

`hubDeps` wires the prime's `shuttlePair` as `Prime`, through `newShuttlePair`.
A prime that fails to wire is logged at Warn and left nil, so `pause` and `resume` behave as today (D4).
It also wires `Processes: proc.ListProcesses`, `HubPath: location.HubPath` and `Executable` from `os.Executable()`, an empty path when that fails, which keeps nothing.
`docs/overview.md`'s tender bullet says what `lyx tender status` shows.

`fakePair` gains a `states map[string]shuttleengine.SessionState` field and its `State` method, which answers the session's scripted state or idle-done, and `newPair` initialises the map.
`env` gains `prime *fakePair`, `processes []proc.Process`, `processErr error`, `hubPath` and `executable`, which `env.deps` passes on.
`env.deps` sets `Deps.Prime` only when `e.prime` is not nil, since a nil `*fakePair` in the interface would read as a prime and panic every existing row that sets none.
`fakePair` also gains `stateErr map[string]error`, which fails `State` for a key present.

`TestStatus` keeps its scenario and gains rows as subtests:
- sessions: pair `p1` under a slug entry with `a` busy, pair `p2` uncovered with `b` idle-stalled `api-error`, and a prime with the orch's session.
  The sessions list all three with their state, cause and since, `Paused` true for `a` only.
  With a hub entry live instead, every session reads `Paused`, the prime's included.
- stale: a deleted `lyx` under `/home/u/go/bin` and one under `<hub>/x/.dev-bin`, both with a cwd inside the hub, are listed with pid, start time and argv.
  A current `lyx`, a deleted `other`, a deleted `lyx` whose cwd is the sibling `<hub>-other`, and one whose cwd is outside the hub are omitted.
  A deleted `lyx` whose cwd is a deleted directory inside the hub is listed.
- failed readings: `stateErr` fails `b`'s state, which is listed `unknown` with the error as its cause; a `processErr` that is not unsupported is reported in `Stale.Error`; the rest of the report still reads.
- unsupported: `processErr` wraps `errors.ErrUnsupported`; `Stale.Supported` is false, and the sessions and live entries still read.
- empty: no entry, no session and no process; `Live`, `Sessions` and `Stale.Processes` are empty, not nil, and `Stale.Supported` is true.

`TestPauseAndResume_WriteLiftAndRefuseOverTheHub` asserts the `status` envelope carries `sessions` as an empty array and `stale` with `supported` true, over a hub with no agents running.

**ImpactSummary:** `pauseengine`'s Pair interface gains one method with its two implementers, the CLI's shuttlePair and the test fake, adapted; Status gains two read-only parts, and pause and resume are untouched.

**Commit:** 9: tender status: every live session in the hub and the processes running a replaced binary
