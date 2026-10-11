# Card 4 — reed-session-filter

**Create:**
- `internal/reedengine#reedSessionFilter`
- `internal/reedengine#listSessionsArgv`

**Edit:**
- `internal/reedengine#ListSessions`
- `internal/reedengine#listSessionsVia`
- `internal/reedengine#endSessionByNameVia`
- `internal/reedengine#switchClientVia`
- `internal/reedengine#Engine.classifyRecordedSessionLocked`
- `internal/reedengine#Engine.Down`
- `internal/reedengine#Engine.sessionlessSocketHolderPersists`
- `internal/reedengine#Engine.ensureServerAndSessionLocked`
- `internal/reedengine#statusFormatSessions`
- `internal/reedengine#TestListSessions`
- `internal/reedengine#TestSwitchClientVia_IssuesOnlyTheThreeCommands`
- `internal/reedengine#TestBoot_TmuxConf`
- `internal/reedengine/doc.go`

**Uses:**
- `internal/reedengine#tmuxConfClaimedOption`
- `internal/reedengine#reedSessionOption`

**Intent:** Make every reed read that lists, counts or cycles a server's sessions see only the sessions that count as reed's (D8), so a session an operator file creates never keeps the hub server alive and is shown by no reed surface.

`reedSessionFilter` is the tmux format that is true for a session that counts: on a server whose `@lyx_tmux_conf_claimed` is `sourced`, one carrying `@lyx_reed_session`; on any other server, every session.
Both are read as `#{@...}` in a format, which resolves the server option and the session option alike (probed), so a server started before this change, which carries no claim, keeps today's rule, and so does a server whose starter sourced nothing.
`listSessionsArgv` returns `list-sessions -f <filter> -F <format>` off Windows and `list-sessions -F <format>` on Windows, where no boot ever claims and psmux's `-f` support is unverified.
Both live in `internal/reedengine/tmuxconf.go`.

Every `list-sessions` argv in the package is built through `listSessionsArgv` with `runtime.GOOS`: `listSessionsVia`, so `ListSessions` filters for the watchdog daemon's discovery and the pair teardown; `endSessionByNameVia`'s and `Down`'s last-session check, so the teardown paths kill the server, and a file-created session with it, when no reed session remains; `sessionlessSocketHolderPersists` and the zombie check in `ensureServerAndSessionLocked`, so a server holding only file-created sessions counts as session-less; `classifyRecordedSessionLocked`; and `switchClientVia`, so the next and previous session switch never lands on a session that does not count.
`ListSessions`'s doc comment names the filter in place of the identical bare invocation it describes.
`statusFormatSessions` wraps each of the `#{S:}` loop's two bodies in `#{?<filter>,<body>,}`, so the status bar's session row skips a session that does not count; a conditional in a loop body is evaluated per session (probed).

`TestListSessions` asserts the one call's argv against a literal, beside its existing capture check: `list-sessions -f <reedSessionFilter> -F #{session_name}` off Windows and `list-sessions -F #{session_name}` on Windows.
It also gains two literal rows over `listSessionsArgv`, in the shape of `TestServerSpawnArgv`'s `goos` rows, one for `linux` and one for `windows`, so both halves of the split are pinned on every host.
`TestSwitchClientVia_IssuesOnlyTheThreeCommands`'s `wantArgv` rows carry the filtered `list-sessions` argv as literals, never built through `listSessionsArgv`.
`TestBoot_TmuxConf` gains the subtest for a file that runs `new-session -d -s operator`: after the boot `ListSessions` returns the reed session alone, the expansion of `#{T:status-format[1]}` against the reed session names no `operator` button, and after the worktree's `Down` the server is gone, which `IsNoServer` on a `ListSessions` error shows; the `switchClientVia` argv row stands in for the switch, since the PTY-attached switch harness is linux-only.
Every other test of these call sites scripts the fake by verb and is unaffected.

`doc.go`'s "Operator config" paragraph gains the D8 rule: which sessions count, that every session read in Go and in a tmux format applies it, that teardown ends the server with the last reed session, and the bound that a file-created session is hidden from reed and lives until the server ends.

Bound: the filter lets the server be killed while a file-created session lives and hides that session from reed, and nothing else; a reed session still keeps the server alive, and on a server without a `sourced` claim the filter admits every session, so Windows and pre-change servers behave as today.
A multiplexer that rejects the format in the status bar degrades the session row, non-fatally like every geometry pin.

**ImpactSummary:** Every `list-sessions` call in one package plus one status-bar format; the two engine-less exports change their listing for `internal/reedcli` and `internal/pairteardown` without a signature change, and the argv on Windows is byte-identical to today's.

**Verify:** lyx gate test --tags tmux ./internal/reedengine
