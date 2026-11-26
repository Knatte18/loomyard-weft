# Reed reads each client's own session, and shellguard refuses the session bus

`reed-client-and-bus-guard` · Task · bug, reed, process · running · Darn-Write

Bundled from the notes below; each part keeps its note's text.

# reed switch and a button's {strand} read the most recent client's session, not the clicker's

`switchClientVia` (`internal/reedengine/switch.go`) reads the clicking client's `#{session_id}` and `runButtonVia`'s strand lookup (`internal/reedengine/buttonclick.go`) reads `#{@lym_strand_name}`, each with `display-message -p -c <client>` and no `-t`.
Run from outside tmux, tmux expands that format against the most recently active client's session, so on a server with clients in several sessions `reed switch` can step from another client's session and a `{strand}` button can name another session's agent.
The RC click had the same defect and now reads its row of `list-clients -F`, which tmux expands per client; `TestClientAgentPane_ReadsEachClientsOwnSession` shows the two-client setup a test of these needs.

# Shellguard rows for the session bus

The study `docs/research/platform/sandbox-without-new-kernel-2026-11-23.md` (`57d15cc02`) ranks shellguard PreToolUse rows first:
refuse `systemd-run`, `busctl --user`, `dbus-send --session` and `gdbus` in run sessions, and `tmux -S` or `-L` aimed at another hub's server.
Raw `tmux` and `script` are already refused in run sessions (`33cbce982`); no row covers the bus commands.
Keep lym's own use of the bus, such as the strand pane's memory-capped scope (`10ae7e112`), working: the rows apply to agent commands only.
