---
round: 3
focus:
  - "Config shape of a `buttons` group in `_lym/discussion/decision-record.md` D2: check which layers (reed base and worktree files, desktop's hub file, desktop's machine layer) load, warn or refuse a group written as a mapping rather than a sequence, against `yamlengine.fillMapping` (`internal/yamlengine/fill.go`), `MissingKeys` (`internal/yamlengine/reconcile.go`) and `ValidateMachine` (`internal/configengine/layers.go`). Settle whether D2 and its acceptance criterion state the behavior of each layer correctly."
  - "Exit status of `lym reed button` in D7: tmux `run-shell` reads the exit status of the pinned shell line, which starts with the pinned binary's absolute path (`internal/reedengine/panebin.go`, `composeFollowCommand`). Settle whether D7 keeps view mode shut when that binary is gone, the verb panics or the verb is killed, on both the click line and the confirmed line."
  - "Run directory of a task session's button commands in D7 and D8: settle whether the stated reasons match `lym batten cancel`'s behavior (`internal/battencli/cancelcmd.go`, `internal/battencli/refusal.go`) and D8's `agent` entry, and whether the departure from the board entry's design is recorded under `## Auto-mode assumptions`."
  - "Trust Bound of D7: for a direct `lym reed button` call without `--confirmed` on an entry with `confirm`, settle whether the identity claim and the Bound state who the entry runs as once the operator answers `y`, given `stripAgentNameEnv` in `internal/reedengine/lifecycle.go`."
---

Each entry restates the sites it depends on.
