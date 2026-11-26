# Read each client's own session; refuse the session bus in agent shells

## Reed reads the clicking client's own session

`reed switch` and a status-bar button's `{strand}` read the clicking client with `display-message -p -c <client>`.
Run from outside tmux, that call expands its format against the most recently active client's session.
With clients in several sessions, `reed switch` could step from another client's session,
and a `{strand}` button could name another session's agent.

Both now read the client's own row of `list-clients -F`, which tmux expands per client.
The RC click already read it that way, and the three share `readClientFormatsVia` (`internal/reedengine/clientrow.go`).
A switch for a client that is not attached now fails with `no client "..." is attached`.

## Shellguard refuses the session bus and tmux aimed at a hub server

The user session bus, and a hub's tmux server, let an agent command start work outside lym's process tree
and its memory caps, or reach a hub's agents directly.
Two new rows close these routes for every run session, the orch included, with no config key:

- `session-bus` refuses `systemd-run`, `busctl --user`, `gdbus` and `dbus-send`.
- `hub-tmux-server` refuses `tmux -L` or `-S` aimed at a server whose name or socket file name opens with `lym-`.

lym's own use of the bus, such as a strand pane's memory-capped scope, runs from Go and never meets a row.

## Design decisions

- `dbus-send` talks to the session bus unless told `--system`, so the row refuses more than `dbus-send --session`.
  A call passes only when `--system` is reached through unquoted words alone and no `--session`, `--bus`
  or `--peer` flag accompanies it.
  A quote, backslash, tab, redirect or substitution before `--system` therefore gets the call refused.
- `gdbus` is refused whole, whichever bus it names.
- `hub-tmux-server` refuses a session's own hub server as well as another hub's, since a session reaches its hub
  through lym's verbs.
  It matters for the sessions whose launch spec lifts the raw `tmux` and `script` deny.
  The sampler stencil now tells such a session never to open a trial server's name with `lym-`,
  and a test ties the row's prefix to `reedengine.ServerName`.
- Both rows are guardrails, not barriers.
  A launch behind a script file, `eval` or a variable passes.
  So do `systemctl --user`, the system bus, a hub socket reached through `TMUX` or a symlink,
  and a quoted socket path with a space in a directory name.

## Merge danger

**Door:** two-way
**Blast radius:** shellguard
