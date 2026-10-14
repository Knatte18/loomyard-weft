# Support log — reed-window-follow

## Interview

A darn run holds no interview; the board entry's scope, written by the operator, is the whole statement of the change.

## Rejected alternatives

- Track each client's window with grouped sessions or `switch-client` per client: every client of one reed session shares its current window, so the session-wide `select-window` already moves each attached client.
- Mark the window a viewer was moved to by reed with a window option and follow only from a marked window: it would add a Go decision on a display option, and the newest live agent window already identifies where reed last put the viewer.
- Follow from every live agent window: it would pull an operator off a window they chose by hand, which the scope forbids.
- Identify the Driver window by the window name `driver`: names are display only; the strand's role is the stable identity.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Is the Driver window identified by role or by window name? Self-pick: by the bound strand's role through `agentname.MatchesRole`. Resolved.
- Q: Does a strand joining an existing shared group window move the viewer? Self-pick: no, it opens no new window, as today. Resolved.
- Q: Does a viewer on the newest agent window by hand count as following? Self-pick: yes, it is indistinguishable and moving it costs one switch. Resolved.
