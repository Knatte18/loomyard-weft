```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I traced each function and branch the diff adds or changes with concrete inputs.

**Cases run**

- `readClientFormatsVia` (`internal/reedengine/clientrow.go:182`)
  - Output `"c0\ttst:orch\nc1\t\n"` with client `c1`: `TmuxCmd.output` does not trim (`overlay.go:138-154`), so the lines are `c0\ttst:orch`, `c1\t` and `""`. The row `c1\t` splits into `["c1",""]`, two fields for one format, and returns `[""]`. **Correct.**
  - Empty trailing format on the last row, with or without the final `\n`: same split. **Correct.**
  - A row ending in `\r`: trimmed before the split. **Correct.**
  - Client absent, or empty output: `no client %q is attached`. **Correct.**
  - Too few fields: `unreadable list-clients row`. **Correct.**
  - tmux failure: returns nil and the wrapped `list clients: %w`. **Correct.**
- `buttonStrand` (`buttonclick.go:248`)
  - An error leaves `fields` nil, and `err != nil ||` short-circuits before `fields[0]`, so there is no panic. **Correct.**
  - An empty strand option shows the reattach message. **Correct.**
  - A name with surrounding spaces is trimmed. **Correct.**
  - Confirmed or answered clicks are unchanged. **Correct.**
- `switchClientVia` (`switch.go:42`)
  - Rows `"/dev/pts/3\t$0\r\n"`: `current[0]` is `$0`, so the step goes to `$2`. **Correct.**
  - The old `TrimSpace` is gone, but the `\r` trim covers line endings, and `session_id` holds no spaces. **Correct.**
  - An empty session id leads `nextSessionID` to return its error. **Correct.**
  - A list failure gives `read the session of client …: list clients: …`. **Correct.**
- `clientAgentPaneVia` (`remotecontrol.go:36`)
  - An empty pane id gives `shows no active pane`. **Correct.**
  - Four fields map to `PaneID`, `StrandName` and `RemoteControlOn`. **Correct.**
  - An unreadable row or a client that is not attached gives the right error, under a new `reed: read the active pane of client %q:` prefix. The behaviour is correct, but the wording changed (finding 3).
- `sessionBusCommandPattern` (`rows.go:412`)
  - Denied as intended: `busctl call --user x`, `busctl --user`, `systemd-run --user`, `/usr/bin/gdbus`, and `dbus-send` with no flag.
  - Let through as intended: `busctl status`, `systemd-runner`, `busctlx --user` (busctl must be followed by whitespace), `busctl --user-x` (`--user` needs a word end), and `ls --user` after `&amp;&amp;` (the argument scan stops at `&amp;`).
  - All **correct**.
- `hubTmuxServerCommandPattern` (`rows.go:434`)
  - Denied as intended: `-L lym-x`, `-Llym-x`, `-S /tmp/tmux-1000/lym-x`, a quoted `-S "…/lym-x"` and `-u -L lym-x`.
  - Let through as intended: `-S /tmp/lym-x/sock` (`/` is not a terminator, so the match fails after backtracking) and `tmux ls &amp;&amp; grep -L lym-x`.
  - Three tmux spellings are not denied (finding 1).
  - No false denial: other tmux subcommands' `-L` and `-S` take a direction or a line number, never `lym-…`.
- `attachOneClientPerSession` and the tier-2 tests
  - In `TestSwitchClient_StepsFromTheClientsOwnSession`, the old read takes the session of the client attached last, `third`. "Next" from there wraps to the booted session, where the moving client already is, so the check that it reached `second` would fail. The test tells old from new. **Correct.**
  - In `TestRunButton_StrandNamesTheClickingClientsOwnPane`, the old read gives both clicks the other session's name, so the first iteration would fail. **Correct.**

**Findings**

1. **LOW**, design, `internal/shellguard/rows.go:434-438` (`hubTmuxServerCommandPattern`).
   - Clustered flags are not denied: `tmux -uL lym-hub-x ls` and `tmux -2S /tmp/tmux-1000/lym-hub-x ls`, both valid getopt spellings. The pattern requires `[[:space:]]+-L` or `-S` as a word of its own.
   - A quoted socket path with a space in a directory is not denied: `tmux -S "/tmp/my dir/lym-hub-x" ls`. `pathChars` excludes whitespace even inside the quote.
   - The row comment lists script, eval, variable, `TMUX` and symlink as ways past the row, but not these two.
   - Evidence: `rows.go:435-438` (pattern text) and `rows.go:660` (row comment).
2. **NIT**, design, `internal/reedengine/remotecontrol.go:38-41`.
   - `clientAgentPaneVia`'s errors are now `reed: read the active pane of client "X": no client "X" is attached`; before they were `reed: no client "X" is attached`. The client is named twice.
   - `_lym/landing/summary.md` says "Its argv and errors are unchanged".
   - `remotecontrol_test.go:41-44` still passes, because it matches substrings.
   - The one production caller, `internal/orchcli/remotecontrol.go:195`, does not parse the text.

Tool calls:

1. Grep for the callers of `ClientAgentPane` and `clientAgentPaneVia`, plus the `remotecontrol_test.go` error rows at HEAD and at the base. My context lacked the error assertions and the production caller.
