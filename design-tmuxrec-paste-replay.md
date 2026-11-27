# The recorded tmux kit cannot replay a pasted call, so no scenario that pastes runs over it

`tmuxrec-paste-replay` · Note · enhancement, testkit, reed

## Seen

The landed recorded tmux kit (`internal/testkit/tmuxrec`) neither records nor matches a call's stdin.
Each paste names a fresh random buffer in its arguments, so a replayed paste matches no recorded one and fails as unanswered.
No current recording holds a `load-buffer` call, so the PR-Review session tests pass; a scenario that sends text to a pane cannot use the kit.

## Idea

Record and match stdin, and match a paste's buffer name by pattern rather than literally, with a recording that pastes and a replay test over it.
