# Card 4 — slackengine-files

**Create:**
- `internal/slackengine/doc.go`
- `internal/slackengine#Geometry`
- `internal/slackengine#Presence`
- `internal/slackengine#PresenceTerminal`
- `internal/slackengine#PresenceSlack`
- `internal/slackengine#ReadPresence`
- `internal/slackengine#WritePresence`
- `internal/slackengine#Setup`
- `internal/slackengine#ReadSetup`
- `internal/slackengine#WriteSetup`
- `internal/slackengine#ReadToken`
- `internal/slackengine#WriteToken`
- `internal/slackengine#RequireConfigured`
- `internal/slackengine#RelayState`
- `internal/slackengine#ReadRelayState`
- `internal/hubgeom#SlackGeometry`
- `internal/slackengine/testmain_test.go`
- `internal/slackengine/files_test.go`
- `internal/slackengine_test#TestPresence`
- `internal/slackengine_test#TestCredentialFiles`
- `internal/hubgeom#TestSlackGeometry`

**Edit:**
- `pattern/PATTERN-cwd-resolution.md`

**Extends:**
- `internal/hubgeom/doc.go`

**Uses:**
- `internal/hubgeom#CreelGeometry`
- `internal/fabricengine#HubScratchDir`
- `internal/fabricengine#RunProcDir`
- `internal/fabricengine#ReadShortname`
- `internal/lymdirs#SlackDirName`
- `internal/githubclient#CredentialDir`

**Intent:** Create `internal/slackengine`, the one package that calls Slack (`PATTERN-agent-addressing` as card 1 words it), with the files every later Slack card reads, and the hub teller of its geometry.

`slackengine` is told its `Geometry` and derives no path (`PATTERN-cwd-resolution`, `PATTERN-lym-files`); it imports no `*cli` package.
`pattern/PATTERN-cwd-resolution.md`'s Bound packages list gains `slackengine`.
`Geometry` holds `HubPath`, `Shortname`, `Dir` (`<hub scratch>/slack`), `TokenPath`, `SetupPath`, `PresencePath`, `CursorPath`, `OutboxPath`, `RelayStatePath`, `LockPath`, `LogPath`, `RunProcDir` and `LymExecutable`.
`hubgeom.SlackGeometry`, in a new `internal/hubgeom/slackgeom.go`, fills it as `CreelGeometry` does: the shortname through `fabricengine.ReadShortname` (a missing record is the same error `CreelGeometry` returns), `Dir` as `filepath.Join(fabricengine.HubScratchDir(hub), lymdirs.SlackDirName)` with the file names as named constants of slackengine, `TokenPath` as `slack.json` in `githubclient.CredentialDir()` (an unresolvable directory is an error naming `HOME`, `XDG_CONFIG_HOME` or `LOCALAPPDATA`), `RunProcDir` as `fabricengine.RunProcDir(hub)` and `LymExecutable` from `os.Executable`.

The files, each written through `internal/state` (`PATTERN-tolerant-records`), each with its lock path beside it under `Dir`'s locks as `lock` names one:
- Token (D2): `{"token": "xoxb-…"}` at `TokenPath`, written atomically by `WriteToken` with the directory created 0700 and the file 0600; `ReadToken` answers found false for a missing file and an empty token.
  No config template, layer or env file holds it.
- Setup (D2): `Setup{ChannelID, OperatorID}` at `SetupPath`, written by `WriteSetup` with `Dir` created 0700 and the file 0600.
- Presence (D6): `{"presence": …, "changed_at": …}` at `PresencePath`; `ReadPresence` answers `PresenceTerminal` for a missing file, and an unknown value is an error naming the file; `WritePresence` reports whether the stored value changed, so a caller sends notices only on a change.
- Relay state (D3, D6): `RelayState{PID, StartTime, FirstReadAt, FirstReadRefusal, LastSlackOK, StartFailures, NextStartAt, LastSpawnAt}`, written by the relay and its starter (card 9) and read by `lym slack away` and `status`.
- `RequireConfigured` reads the token and the setup file and refuses a missing one: "slack: no Slack token on this machine; way forward: run `lym slack login` from the operator's terminal" and "slack: the hub's Slack channel and operator are not set; way forward: run `lym slack setup --channel <id> --operator <user-id>` from the operator's terminal" (D2, D3).

`internal/hubgeom/doc.go`'s list of tellers gains `SlackGeometry`, the slack engine's teller.
`internal/slackengine/doc.go` documents the package: the Slack transport of the hub relay, the files under the hub's `.lym/slack/` and the machine token file, and that it holds the only Slack calls.
`internal/slackengine/testmain_test.go` is `untagged`: `TestMain` runs through `tmuxkit.Main`.
`internal/slackengine/files_test.go` is tier0, on `t.TempDir()` paths:
`TestPresence` rows — a missing file reads `terminal`; a write of `slack` reads back and reports changed; a second write of `slack` reports unchanged; an unknown stored value is an error.
`TestCredentialFiles` rows — a token written reads back, its file mode is 0600 and its directory 0700 (off Windows); a missing token file reads found false; a setup written reads back with mode 0600; `RequireConfigured` with no token, and with a token but no setup file, refuses naming `lym slack login` and `lym slack setup` respectively, and with both returns them.
`TestSlackGeometry` (tier0, in `internal/hubgeom`'s existing tier0 test file `hubgeom_test.go`) writes a shortname record under a temp hub and asserts every path lies under `<hub scratch>/slack` but the token path, which lies in the redirected credential directory, that a hub with no shortname record is the creel-shaped error, and that with `HOME`, `XDG_CONFIG_HOME` and `LOCALAPPDATA` all cleared through `t.Setenv` the error names them; it redirects the credential directory with `t.Setenv`, process-global state, so it does not run in parallel.

**ImpactSummary:** Only `internal/hubgeom/doc.go`'s teller list and `PATTERN-cwd-resolution`'s Bound packages list grow by one name each; everything else is new code in the new package and one new hubgeom file.
