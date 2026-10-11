---
round: 2
focus:
  - "In decision-record.md D3 and D2, settle whether the rule for which boot sources the operator's tmux config file is decidable from reed's code: `ensureServerAndSessionLocked` in `internal/reedengine/lifecycle.go` runs `serverSpawnArgv` whenever the worktree's session is missing, including a second worktree joining a running server, and the op lock `reed.lock` is per worktree, so concurrent boots are not serialized. Check that the Decision states how a boot learns it started the server, what happens under concurrent boots, and that an acceptance criterion covers it."
  - "In decision-record.md D5 and its acceptance criteria, settle whether every pinned option is achievable in the stated order. On tmux 3.6, sourcing a file with `set -g destroy-unattached on` into a detached session destroys that session at once, so a pin applied after `source-file` cannot save it, and a per-session pin leaves later joining sessions on the file's global value. Check that each acceptance criterion about a pin can pass under the chosen ordering and scope."
  - "In decision-record.md D4 and its acceptance criteria, settle whether the stated tmux behavior on a bad config file is accurate. On tmux 3.6 a parse error applies none of the file's commands, while a command failing at run time leaves the other commands applied. Check that the rationale and the criteria distinguish the two."
  - "In decision-record.md, settle whether the treatment of sessions not created by reed is decided. Reed kills the hub server only when `list-sessions` is empty, so a session created by the operator's file keeps the server alive after the last worktree goes down. Check that a Decision states how reed treats such sessions at boot and teardown, with a stated bound, and that the next-server-start rule for file changes still holds."
---

The focus lists question-level checks only; it caps no severity and pre-states no verdict.
Each entry restates its own sites and claims so the reviewer needs no earlier round.
