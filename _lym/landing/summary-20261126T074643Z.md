# Reed's RC click reads the clicking client's own session

From issue #214.

A click on reed's RC button runs `lym orch rc-on --client <name>`, which reads the clicking client's active pane through `reedengine.ClientAgentPane`.
That read ran `tmux display-message -p -c <client>` with no `-t`.
Run from outside tmux, display-message expands its format against the pane tmux picks as current, which is the most recently active client's session, not the `-c` client's.
On a server with clients in several sessions every click could read the same pane, such as the orch's,
so a click in a task session could queue remote control into the orch or be refused as already on.

`clientAgentPaneVia` now runs `list-clients -F '#{client_name}\t#{pane_id}\t#{@lym_strand_name}\t#{@lym_rc}'` and takes the clicking client's row.
tmux expands each list-clients row against that client's own session, so the pane id and both pane options come from the pane the clicker sees.

## Design decisions

- `list-clients` over `display-message -t <client's session>`: it needs one tmux call and no session-name target, which an exact-match target would have to quote.
- A client absent from the listing is now its own error, `no client "<name>" is attached`; a listed client with an empty pane id keeps the `shows no active pane` error.

## Tests

- `TestClientAgentPane_ReadsEachClientsOwnSession` (tier 2, live tmux) attaches one client to each of two sessions and checks each client's read names its own session's pane and strand name.
  On the old code both reads name the last-attached client's session, so the first client's read fails.
- `TestClientAgentPaneVia` (tier 0) pins the new argv, that other clients' rows are skipped, and the unlisted-client error.

## Follow-up

`reed switch` (`switch.go`) and a button's `{strand}` lookup (`buttonclick.go`) run the same `display-message -p -c <client>` with no `-t`.
They are left as they are here and filed as board note `reed-switch-wrong-client`.

## Merge danger

**Door:** two-way
**Blast radius:** reed
