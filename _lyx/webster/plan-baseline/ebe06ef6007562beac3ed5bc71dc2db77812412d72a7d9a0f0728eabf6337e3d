# Card 5 — creelcli-go-client

**Create:**
- `internal/creelcli#GoClient`
- `internal/creelcli#NewProgramClient`
- `internal/creelcli#NewStrandClient`
- `internal/creelcli#newGoClient`
- `internal/creelcli#goCallTimeout`
- `internal/creelcli#GoClient.Shortname`
- `internal/creelcli#GoClient.Post`
- `internal/creelcli#GoClient.ReadAll`
- `internal/creelcli#GoClient.Inject`
- `internal/creelcli#GoClient.InjectCommand`
- `internal/creelcli#GoClient.Log`
- `internal/creelcli#GoClient.Hold`
- `internal/creelcli#GoClient.Release`
- `internal/creelcli#GoClient.Withdraw`

**Edit:**
- `internal/creelcli#logEntryFields`
- `internal/creelcli#logCmd`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli/doc.go`

**Intent:**
`internal/creelcli` exposes the Go-side calls other modules need, so no caller builds a creel request by hand (decision record D10).

`GoClient` wraps a `creelengine.Client` for the hub of a location.
`NewProgramClient` signs as the program `program` names, an upper-case name such as `BATTEN` that each sending module owns as a constant, formed into the full name with the hub's shortname through `agentname.FormatProgram` (`PATTERN-agent-name`); its claim carries no strand name, whatever `$LYX_STRAND_NAME` holds, so a program client never claims a strand.
`NewStrandClient` claims the strand `strandName` names and no program.
Both go through `newGoClient`, which refuses a location outside a hub naming the way forward, resolves the creel geometry through `hubgeom.CreelGeometry` and the lyxtemp root as `runVerb` does, starts the daemon through the package's spawn seam when none answers, and builds no connection until the first call.
Every call passes `goCallTimeout`, which card 4 made bound every request, so a caller under a lock waits at most that long per call (decision record D7's bound).
`Post`, `Inject`, `InjectCommand`, `Hold`, `Release` and `Withdraw` send the matching verb and answer its id or result; `ReadAll` reads every unread message of the caller without waiting and hands them to `handle` before the client acknowledges them, so a handler error leaves them unread; `Log` answers the journal entries matching the filter.
Bound: this adds no verb and no CLI flag; a program claim still reaches the daemon only from Go code, and the daemon's identity rules decide who the caller is.

`logEntryFields` renders the `command` and `argument` keys card 3 added to `LogEntry`, so `lyx creel log` shows a command inject by its command name and argument, and `logCmd`'s help says so.

Tests:
`TestCreelCLI` gains rows over its in-process daemon: a program client posts to a strand and the strand reads the mail signed by the program; a program client's command inject to the rig's orch strand is typed through the fake delivery's `Command`, and `lyx creel log --id` prints its command and argument; a program client's `ReadAll` with a failing handler leaves the reply unread and a second `ReadAll` hands it over; a program client's withdraw removes its queued inject; `NewProgramClient` signs as `<shortname>:<program>` with `$LYX_STRAND_NAME` set to a strand; `NewProgramClient` on a location outside a hub is refused naming the way forward.
`internal/creelcli/doc.go` describes the Go client and creel's typed refusal code: the refusal of a strand claim from a process in no strand pane reaches a Go caller as an error matching `creelengine.ErrStrandClaimOutsidePane`, while the CLI prints it as any refusal.

**ImpactSummary:** New exported Go client in creelcli plus two log fields in one CLI rendering; existing verbs and their output are unchanged apart from the added keys.
