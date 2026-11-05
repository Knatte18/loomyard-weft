---
round: 2
exclude_lenses: []
focus:
  - "Relay sender identity: `proc.Detach` in `internal/proc/proc_linux.go` only sets `Setsid`, and creel's `paneOfPeer` in `internal/creelengine/identity.go` walks the peer's parent pids. Settle whether the decision record's account of how the relay gets its `<shortname>:SLACK` claim holds when `lym slack away` or `lym slack start` runs from a non-orch strand's pane, and whether its acceptance criteria can pass."
  - "Cursor durability: settle whether a relay that starts, handles no message and dies loses any message written while it was down once it restarts, given where the record says the cursor is persisted."
  - "Relay lifecycle versus its restarters: `internal/fabriccli/anchor.go` calls `deps.EndDobbies(dobbies, runproc.RolePush, runproc.RoleSync)` and restarts only dobbies; the prime dobby starts a down relay while presence is `slack`. Settle what a prime anchor change and `lym slack stop` leave running, in each presence state, and whether the record states and bounds it."
  - "Mail retention and posting: `internal/creelengine/journal.go` `retain` carries unread messages and a program's read forward. Settle whether every bound the record states on how long SLACK mail waits, and on how a failed or rate-limited post is retried, holds against that behavior and drains without reposting a message."
  - "Reminder wait start: shed appends a history entry only when a call ends or a goto writes one, and a pause or resume rewrites status with history unchanged (`internal/shedengine/run.go` `persistTransient`). Settle whether the record's definition of the wait start and its bound hold across a pause, a resume and a dobby restart."
  - "Attack surface of the new verbs: for each new `lym slack` verb and Slack text command, settle whether the introducing Decision states who may run it, what it lets through and what bounds it."
---

Every lens ran in round 1 with MEDIUM findings, so none is excluded.
