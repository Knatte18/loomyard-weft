# A paste over tmux message size fails whole, with no chunking

`paste-size-limit` · Note · enhancement, shuttle, reed

Seen at pane-inject-safety PR-Gate (PR #548): the one-command set-buffer + paste-buffer + Enter path sends the text in one tmux command, so a text over tmux's command size limit fails the whole send.

Idea: load the buffer in chunks (or from a file with `load-buffer`) before the single paste-and-Enter step, and test a text above the limit.