# Card 8 — reed-button-verb

**Create:**
- `plan:internal/reedengine#ButtonRequest` -> `type ButtonRequest struct`
- `plan:internal/reedengine#RunButton` -> `func RunButton(req ButtonRequest)`
- `plan:internal/reedengine#buttonDeps` -> `type buttonDeps struct`
- `plan:internal/reedengine#runButtonVia` -> `func runButtonVia(cmd TmuxCmd, req ButtonRequest, deps buttonDeps)`
- `plan:internal/reedengine#runButtonScript` -> `func runButtonScript(shellPath, flag, script, dir string) (output string, exitCode int, err error)`
- `plan:internal/reedengine#buttonRunHash` -> `func buttonRunHash(run string) string`
- `plan:internal/reedengine#tmuxQuote` -> `func tmuxQuote(s string) string`
- `plan:internal/reedengine#composeConfirmedButtonLine` -> `func composeConfirmedButtonLine(sh shell.Shell, exe string, req ButtonRequest, dir, hash, strand string) string`
- `plan:internal/reedengine#TestRunButtonVia` -> `func TestRunButtonVia(t *testing.T)`
- `plan:internal/reedengine#TestRunButton_RealTmux` -> `func TestRunButton_RealTmux(t *testing.T)`
- `plan:internal/reedcli#reedCLI.buttonCmd` -> `func (c *reedCLI) buttonCmd() *cobra.Command`
- `plan:internal/reedcli#validateButtonFlags` -> `func validateButtonFlags(req reedengine.ButtonRequest) error`

**Edit:**
- `internal/reedengine/doc.go`
- `internal/reedcli#Command`
- `internal/reedcli#TestRunCLI_Switch_RefusesBadFlagsAndNeedsNoRepository`
- `cmd/lym#TestSkipStencilSeed_HonoursTheAnnotation`

**Extends:**
- `internal/reedcli/doc.go`

**Uses:**
- `plan:internal/reedengine#buttonSpec`
- `plan:internal/reedengine#buttonSpecEntry`
- `plan:internal/reedengine#buttonSpecOption`
- `plan:internal/reedengine#buttonRangePrefix`
- `plan:internal/reedengine#strandNamePaneOption`
- `internal/reedengine#fillRunPlaceholders`
- `internal/reedengine#fillConfirmPlaceholders`
- `internal/reedengine#newTmuxCmdForSocketPath`
- `internal/reedengine#installFakeTmuxOn`
- `internal/reedengine#newIntegrationEngine`
- `internal/reedengine#startInPTY`
- `internal/reedengine#waitForClientAttached`
- `internal/reedengine#waitUntil`
- `internal/reedengine#mustTmuxOutput`
- `internal/reedcli#reedCLI.followCmd`
- `internal/shell#Shell`

**Intent:**
`lym reed button` resolves a clicked range to the spec entry the session holds, confirms when the entry asks, runs it, logs it and reports a failure (D7).
It reads no config and no reed state: only the session's `@lym_button_spec` and, for `{strand}`, the clicking client's active pane's `@lym_strand_name`.

reedengine (a new file `buttonclick.go`, `composeConfirmedButtonLine` in `panebin.go` beside the other composed lines):
- `ButtonRequest` holds `TmuxPath`, `SocketPath`, `Session`, `Range`, `Client`, and for the confirmed line `Confirmed`, `Hash` and `Strand`.
- `RunButton(req)` is the engine-less function behind the verb, like `FollowSession`: it builds the `TmuxCmd` with `newTmuxCmdForSocketPath` and calls `runButtonVia` with `buttonDeps{run: runButtonScript, executablePath: os.Executable, sh: shell.ForGOOS()}`.
  It returns nothing: every path below ends without an error, so the verb exits 0.
- `runButtonVia`, in order; each "message" is `display-message -c <client> <text>` with every `#` and every `%` in the text doubled, since `display-message` expands both formats and strftime fields, so an output line such as `100% done` or a directory holding `#` shows literally; a failed message is logged at Debug only, as when the client is gone after CANCEL:
  1. Read `show-options -qv -t =<session>: @lym_button_spec` and decode it; a failed read, a value that is not valid JSON and an empty value are each logged at Warn, give the message "lym: the buttons are not rendered for this session; reattach it with lym reed attach" and run nothing, since the attach pre-flight renders the spec.
  2. Find the entry whose index is the number after `lymbtn` in `Range`; no entry at that index (a changed spec) and a suffix that is not a number (a non-button range) each give the message "lym: the buttons changed; click again" and run nothing.
  3. For a `strand` entry: unconfirmed, read `display-message -p -c <client> '#{@lym_strand_name}'`; an empty or failed read gives the message "lym: <label> needs the agent's name, which this pane lacks; reattach the session with lym reed attach and click again" and runs nothing.
     The agent row draws a `{strand}` entry only while the active pane carries `@strand`, so this read is empty only on a strand pane an older lym launched, and the attach pre-flight's re-assert (card 7) sets the name.
     Confirmed, the carried `Strand` must pass `agentname.Parse`, else the message "lym: <label>: the strand name <name> is not a strand name; click the button again from the agent pane" and nothing runs.
     A direct call that is neither confirmed nor reads a name is the same unconfirmed path.
  4. The strand value, when any, fills `{strand}` in `run` through `fillRunPlaceholders(deps.sh, ...)` and in `confirm` through `fillConfirmPlaceholders`.
  5. A run directory that is not an existing directory gives the message "lym: <label>: <dir> is gone; the row re-renders at the session's next attach" and runs nothing, so a confirm entry whose run directory is gone raises no prompt.
  6. An entry with `confirm`, not confirmed: issue `confirm-before -t <client> -p "<confirm> (y/n)" <command>`, the command `run-shell -b <tmuxQuote(line)>`, the line `composeConfirmedButtonLine(deps.sh, exe, req, spec.dir, buttonRunHash(entry.run), strand)` with `exe` from `deps.executablePath`; then return.
     An unresolvable executable is logged at Warn, gives the message "lym: <label> could not raise its prompt; see the lym log in <dir>/.lym/logs" and runs nothing.
     A confirmed call whose `Hash` differs from `buttonRunHash` of the entry's spec `run` gives the message "lym: the buttons changed since <label> asked; click again" and runs nothing; no `run` text crosses tmux's parsing, only the hash.
  7. `deps.run(spec.shell, spec.flag, run, spec.dir)`; a spawn failure or a non-zero exit gives the message "lym: <label> failed (exit <code>): <first output line>".
- `runButtonScript` starts `shellPath flag script` in `dir`, waits for it with no timeout, and returns the combined output and exit code; it logs the spawn and the end, with exit code and combined output, at Info via `logger` (`PATTERN-spawn-observability`), so the output lands in the log of the run directory's anchor, the verb's cwd.
  It sets no caller identity: the click line runs from the tmux server, whose environment reed stripped of `agentname.StrandNameEnv` and `agentname.ParentEnv` at spawn (`stripAgentNameEnv`).
  The command is the operator's action and is not registered in `internal/runproc`.
- `buttonRunHash` is the hex SHA-256 of `run`.
- `tmuxQuote` wraps `s` in double quotes for tmux's command parser, escaping `\`, `"` and `$` with a backslash and doubling `#`, since `run-shell` format-expands its command.
- `composeConfirmedButtonLine` is `sh.Chain(sh.ChangeDir(dir), sh.DiscardOutput(<words>), sh.ExitSuccess())`, the words `sh.Invoke(exe) reed button` then `--socket`, `--tmux`, `--session`, `--range` and `--client` with the request's values, `--confirmed --hash <hash>`, and `--strand <strand>` when non-empty, each value `sh.Quote`d.
  The line opens with the directory change, so the confirmed verb starts in the run directory, because `run-shell` does not format-expand its `-c` start directory (card 9); `tmuxQuote` doubles every `#` of the line, the directory's included, like every other pinned token.
  The forced zero status and the discarded output keep tmux 3.6 from opening view mode on the target pane, whatever the verb does.

reedcli: `buttonCmd` is `lym reed button`, flags `--socket`, `--tmux`, `--session`, `--range`, `--client`, `--confirmed`, `--hash` and `--strand`, annotated like `follow`: `clihelp.AudienceRole` and `clihelp.SkipStencilSeedAnnotation`, no args.
`validateButtonFlags` refuses a missing `--socket`, `--tmux`, `--session`, `--range` or `--client`, and `--confirmed` without `--hash`, each naming the flag, as an error envelope with a non-zero exit, like `follow`'s refusals; only a direct call reaches them, since the pinned and confirmed lines always pass every flag.
Otherwise it calls `reedengine.RunButton` and prints nothing.
`Command` registers it after `followCmd`, and its `PersistentPreRunE` skip list gains `button` beside `switch` and `follow`, with the guard comment saying button is told its socket and session on its flags: the verb resolves no location and loads no config, so a `reed.yaml` that fails to load cannot abort a click (D7).
`reedcli/doc.go` gains its paragraph, and `reedengine/doc.go` a paragraph on `RunButton`.
`cmd/lym/stencilseedgate_test.go`'s file header and `TestSkipStencilSeed_HonoursTheAnnotation`'s doc comment, which name the commands that must carry the annotation, add "lym reed button".

What the verb lets through, and its bound (D7): it runs only the command the session's spec holds at the clicked index.
Any process that can run `lym reed button` against the socket can run a spec entry, `--confirmed` skipping its prompt, under its own caller identity, which adds nothing over running the command itself; `--strand` lets such a caller fill `{strand}` with any name `agentname.Parse` accepts, quoted per value so it stays one word, which likewise adds nothing over that caller running the command itself; a direct call without `--confirmed` raises the prompt on whichever client it names, and a `y` there runs the entry as the operator, which adds nothing over the socket, which can already run commands through `run-shell`.
A process that can write a file defining buttons, or reach the socket, can change what a button runs on the operator's next click, adding only the caller identity; accepted.
Two clicks run the command twice; irreversible entries carry `confirm`.

Tests:
- `TestRunButtonVia` (`tier0`, a new internal test file under `//go:build tier0`) drives `runButtonVia` over a `TmuxCmd` from `newTmuxCmdForSocketPath` with the fake tmux installed on it through `installFakeTmuxOn`, a stub `run` recording its calls and returning a scripted output, code and error, and a stub executable path, one table row per path.
  It reads the Warn and Debug lines through `logcapture`, so it does not call `t.Parallel`, and a comment names the logger's process-global output as the reason.
  Rows: a plain entry runs with the spec's shell, flag, filled `run` and dir and issues no message; a range with no entry at its index and a range `lymbtnx` whose suffix is no number each run nothing and show the changed-buttons message; an unreadable spec option, a spec value `{` that is not JSON and an empty spec each run nothing and show the not-rendered message naming `lym reed attach`; a `{strand}` entry reads `%`-free name `tst:a-task:agent` and runs with it quoted; a `{strand}` entry whose read answers empty, and one whose read fails, run nothing and show the agent-pane message naming `lym reed attach`; a confirm entry issues exactly `confirm-before -t <client> -p "<confirm> (y/n)" run-shell -b "<line>"` with the hash and the line opening with the change to the run directory, and runs nothing; the same with the stub executable path failing issues no `confirm-before`, runs nothing and shows the could-not-raise message; the confirmed call with the matching hash runs; with another hash runs nothing and shows the changed message; a confirmed `{strand}` call carrying `not a name` runs nothing and shows the strand-name message; a confirm value `a#b%c` for `{strand}` reaches the prompt as `a##b%%c`; a removed run directory, on a plain entry and on an unconfirmed confirm entry, runs nothing, issues no `confirm-before` and names it; a run exiting 3 with output `first\nsecond` shows `lym: <label> failed (exit 3): first`; a run exiting 1 with output `100% done` under the label `a#b` shows `lym: a##b failed (exit 1): 100%% done`; a stub `run` returning a spawn error shows the failed message with that error; a failing `display-message` after a failed run is logged at Debug and nothing else fails.
  A second subtest pins `composeConfirmedButtonLine` and `tmuxQuote` for both dialects over values, the run directory among them, holding a space, `'`, `"`, `$` and `#`.
- `TestRunButton_RealTmux` (`tier2 && linux`, in a new test file under that tag) boots a real server through `newIntegrationEngine`, attaches a client in a pty, sets `@lym_button_spec` by hand to a spec whose shell is `/bin/sh`, whose dir is a temp directory with a space in its name and whose entries are `pwd > out`, `echo first; exit 3` and a confirm entry `touch confirmed`, then calls `runButtonVia` with the real `TmuxCmd`, `runButtonScript` and a recorder script as the executable: the first entry writes the run directory into `out`; the second logs its exit code and output (read through `logcapture`, so the test does not call `t.Parallel`, and a comment names the logger's process-global output as the reason); the confirm entry raises the prompt, a `y` typed into the pty runs the recorder, whose recorded arguments carry `--confirmed`, the entry's hash and the session intact; calling `runButtonVia` with those arguments as a confirmed request creates `confirmed`.
  The spec also holds a `{strand}` entry `printf %s {strand} > strand`: with the client on a pane carrying `@lym_strand_name` `tst:a-task:agent`, set by hand, the call writes that name into `strand`, so the real read resolves the clicking client's active pane.
- `TestRunCLI_Switch_RefusesBadFlagsAndNeedsNoRepository` (`tier0`) gains a row per missing required `button` flag and one for `--confirmed` without `--hash`, each naming the flag.
  A row runs `button` with every required flag, a socket path that does not exist and no repository in the working directory, and expects exit 0 and no output, so the verb skips the pre-run's location and config resolution.
- `TestSkipStencilSeed_HonoursTheAnnotation` (`tier0`) gains the row "reed button carries the annotation" over `reedSubcommand(t, "button")`.

**ImpactSummary:** a new role-audience reed verb with an engine-less entry point; `Command` registers one more subcommand; the CLI flag-refusal and stencil-seed annotation tables gain rows; nothing existing changes behaviour.

**Commit:** 8: Add lym reed button, which runs a clicked status-bar button
