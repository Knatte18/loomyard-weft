# Reed's RC click reads the most recent client's session, not the clicker's

`rc-click-wrong-client-session` · Task · bug, reed · done

Imported from [issue #214](https://github.com/Knatte18/loomyard/issues/214).

`ClientAgentPane` (`internal/reedengine/remotecontrol.go:40`) runs `tmux display-message -p -c <client>` without `-t`. tmux then resolves the format against the most recently active client's session, not the clicking client's. On a reed server with four clients in different sessions (one in the prime's orch session, the others in task sessions), it returned the orch session and `mo:orch` for all four. So an RC click in a task session can target the orch's pane instead of the task's agent, and is refused as already on when the orch is marked. Found while diagnosing #212. Fix: pass `-t <client>` (or resolve the client's session first) in `clientAgentPaneVia`.