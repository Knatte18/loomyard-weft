# Read the RC click's pane from the clicking client's own session

An RC click now reads the active pane of the session the clicking client is in.
Before, `ClientAgentPane` ran `tmux display-message -p -c <client>` without `-t`,
and tmux expanded the format against the most recently active client's session.
With clients in several sessions, a click in a task session could target the orch's pane instead of the task's agent,
and was refused as already on when the orch was marked.

Closes #214

## Design

- `clientAgentPaneVia` now runs `list-clients`, which expands each row's format against that client's own session,
  and picks the row whose `client_name` equals the clicking client exactly.
  This takes one tmux call, where resolving the client's session first and then passing `-t` would take two.
- The errors now tell apart a client that is not attached, a row that cannot be read and a client with no active pane.
- A tier-2, Linux-only test attaches two clients to two sessions on a real tmux server
  and checks that each read returns its own session's pane.

## To check by hand

- No hand-run of an RC click is recorded:
  with clients attached to several sessions, click RC in a task session and confirm that the task's agent gets it,
  including while the orch's pane is marked.

## Merge danger

**Door:** two-way
**Blast radius:** reed
