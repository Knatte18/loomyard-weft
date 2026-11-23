---
round: 2
focus:
  - "internal/shuttleengine/session.go Runner.TurnOnRemoteControl: after /remote-control is typed, does any error path (transcript read failure, unconfirmed paste) make creel's delivery retype the command or leave a connected pane unmarked?"
  - "internal/orchcli/start.go relaunchStrand: is the orch's state saved and its process registered before the bounded remote-control connection wait, while the start lock is held?"
  - "internal/shuttleengine/claudeengine/remotecontrol.go: is the session transcript path built through one named constant or helper, with no new repeated `.jsonl` literal, as PATTERN-code-conventions requires?"
  - "Tests added for the remote-control checkpoint, connection read and error paths in internal/shuttleengine, internal/shuttleengine/claudeengine, internal/creelcli and internal/orchcli: does each assert an independent value, and does each pin behavior no other test covers?"
---

Focus is derived from the round's gating findings and its test-coverage gaps.
