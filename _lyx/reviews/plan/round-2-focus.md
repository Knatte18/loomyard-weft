---
round: 2
focus:
  - "Card 7 makes `ensureServerAndSessionLocked` refuse an unresolvable shell in its pre-tmux block on every GOOS but windows, while the shared fixture `newTestEngine` in `internal/reedengine/lock_test.go` sets a nonexistent shell. Settle whether every untagged test that reaches `Up`, `Resume` or `ensureServerAndSessionLocked` through `newTestEngine` or `reviveTestEngine` (`internal/reedengine/revive_test.go`) against a fake tmux is either a card 7 target or given a resolvable shell by the plan."
  - "The Shared Decision `probe-server-bound` in `_lyx/plan/00-overview.md` keeps the capability probe's `list-commands` without `-f /dev/null`, premised on the probe's transient server exiting when sessionless. Settle whether that server reads the operator's `~/.tmux.conf` and can then hold a session that `new-session -f /dev/null` joins, against the decision record's D7 guarantee that reed's own servers read no `~/.tmux.conf`."
  - "Card 5 replaces `watchLoop`'s ticker with a backed-off poll wait in `internal/reedengine/watchloop.go`. Settle whether every existing loop test that stays in poll mode on unchanged ticks, such as `TestWatchLoop_OpenerFailureStaysInPollMode`, still passes under the test ceiling the card sets, and whether the card's stated reason and doc follow-ups match."
---

Focus targets the plan's card 7 and card 5 test interactions and the probe decision; the remaining round 1 findings are small wording or granularity items for the fixer.
