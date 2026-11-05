---
round: 4
focus:
  - "Where the Slack channel id and the operator's Slack user id live and who can write them. Check `_lym/discussion/decision-record.md` D2, D3 (`lym slack status`) and D4 (new-channel path) against `internal/configcli/hubwide.go` and `internal/configcli/doc.go`, which refuse a hub-wide machine-layer write only from a non-prime worktree or a slug-carrying `LYM_STRAND_NAME` and describe themselves as a mistaken-write guard. Settle whether the record's stated bound on repointing the channel or operator id holds for every writer, `lym config slack --layer machine` included, and whether the live relay's per-poll read reaches a repoint without `lym slack setup`'s `conversations.info` and `users.info` checks."
  - "What the pane-ancestry refusal does when a live strand's pane process is unknown. Check D2, D3 (`lym slack start`) and D6 of `_lym/discussion/decision-record.md` against `internal/creelengine/identity.go` (`paneOfPeer`), where a process in no known pane cannot be told from a strand with an unknown pane process and creel refuses. Settle whether the record says which way `login`, `setup`, `start`, `away`, `back` and `stop` go in that window, and what bounds it."
  - "What the hidden `lym slack relay` verb does when run by hand and when its first creel read is refused. Check D3, D4, D6 and D9 of `_lym/discussion/decision-record.md` against `internal/creelengine/identity.go`, where creel refuses a program claim from a non-orch pane. Settle whether the record states the relay's audience, whether it holds its single-instance lock before or after the first creel read, and whether a refused read leaves the lock free for the real relay."
---

Each entry names a site and a question; none caps severity or pre-states an answer.
