# reed switch and a button's {strand} read the most recent client's session, not the clicker's

`reed-switch-wrong-client` · Note · bug

`switchClientVia` (`internal/reedengine/switch.go`) reads the clicking client's `#{session_id}` and `runButtonVia`'s strand lookup (`internal/reedengine/buttonclick.go`) reads `#{@lym_strand_name}`, each with `display-message -p -c <client>` and no `-t`.
Run from outside tmux, tmux expands that format against the most recently active client's session, so on a server with clients in several sessions `reed switch` can step from another client's session and a `{strand}` button can name another session's agent.
The RC click had the same defect and now reads its row of `list-clients -F`, which tmux expands per client; `TestClientAgentPane_ReadsEachClientsOwnSession` shows the two-client setup a test of these needs.