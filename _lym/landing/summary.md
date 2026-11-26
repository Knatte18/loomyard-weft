# Reed reads the clicking client's session; shellguard refuses the bus

`reed switch` and a `{strand}` button now act on the session of the client that clicked.
Before, both read the client's session or pane with `display-message -p -c <client>` and no `-t`.
Run from outside tmux, that call expands its format against the most recently active client's session.
On a server with clients in several sessions, `reed switch` could step from another client's session, and a `{strand}` button could name another session's agent.

Agent shells also lose two ways out of lym's process tree: the user session bus, and a tmux server that belongs to a hub.

## What changes

- `internal/reedengine/clientrow.go` adds `readClientFormatsVia`.
  It reads the client's own row of `list-clients`, which tmux expands against each client's own session.
  It tells apart a failed tmux call, a client that is not attached, and a row it cannot read.
- `switchClientVia` (`switch.go`) reads the client's `#{session_id}` through it, and `buttonStrand` (`buttonclick.go`) reads `#{@lym_strand_name}` the same way.
  The RC click's `clientAgentPaneVia` already read `list-clients`, and now uses the shared helper.
  Its argv and errors are unchanged.
- `internal/shellguard` gains two rows, installed and announced in every session, the orch included (`SwitchNone`):
  - `session-bus` refuses `systemd-run`, `busctl --user`, `dbus-send` and `gdbus` in command position.
    The steer names the operator or the parent as the way forward when work needs the bus.
    lym's own use of the bus, such as the memory-capped scopes in `internal/proc` and `panememory.go`, runs from Go and never meets a PreToolUse row.
  - `hub-tmux-server` refuses `tmux -L lym-…` and `tmux -S <path>/lym-…`, the name every hub's tmux server carries.
    A directory along the socket path that opens with `lym-` is no match.
- Docs: the `reedengine` package doc records the `display-message -c` behaviour among its tmux facts, and the `shellguard` package doc describes the two rows.
- The claude settings golden (`testdata/settings-golden.json`) and the settings tests that count installed rows follow the two new rows.
  The codex engine derives its rows from the table and needs no change.

## Tests

- Tier 0: `TestSwitchClientVia_IssuesOnlyTheThreeCommands` pins the `list-clients` read and adds rows for a client among others, a client that is not attached, and a failed listing.
  `TestRunButtonVia` now feeds a `list-clients` answer whose first row belongs to another client.
- Tier 2, against a real server with two clients in two sessions, the second attached last:
  `TestSwitchClient_StepsFromTheClientsOwnSession` and `TestRunButton_StrandNamesTheClickingClientsOwnPane`.
  Both read the moved client's state through `list-clients -t`, so their checks do not hit the same defect.
  `TestClientAgentPane_ReadsEachClientsOwnSession` now attaches its clients through the shared `attachOneClientPerSession` helper.
- Tier 1: `TestRows_PatternsDenyAndAllow` gains rows for both guards.
  It also checks that `hub-tmux-server` denies the name `reedengine.ServerName` derives for a hub, so the row and the naming cannot drift apart.

## Design decisions

- `dbus-send` is refused whatever its flags, not only with `--session`: it talks to the session bus unless told `--system`.
  `busctl` is refused only with `--user`, since it defaults to the system bus.
- Raw `tmux` is already refused in every session but those whose launch spec sets `AllowRawTmux`, such as the sampler's.
  The `hub-tmux-server` row is what reaches those sessions.
  It refuses any hub's server, the session's own hub included: a static pattern cannot tell this hub's server from another's,
  and a session reaches its own hub through lym's verbs.
- The `lym-` prefix is a constant in `shellguard`, which cannot import `reedengine`.
  The tier-1 check against `reedengine.ServerName` keeps the two together.
- Both rows are guardrails, not barriers: a script file, `eval`, a variable, or a socket reached through `TMUX` or a symlink gets past them, as each row's comment states.

## To check by hand

- No hand run is recorded.
  With two terminals attached to two sessions of one hub, click the switch and a `{strand}` button in the terminal that was not used last.
  Each should act on that terminal's own session.
- In a run session's Bash, `busctl --user status` and `tmux -L lym-x ls` should each be refused with the row's way forward.

## Merge danger

**Door:** two-way
**Blast radius:** shellguard
