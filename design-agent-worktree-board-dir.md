# lyx creates <prime>/.claude/worktrees/_board/.lyx/pause/net when run from a Claude Code agent worktree

`agent-worktree-board-dir` · Task · bug, fabric · running · PR-Gate

Imported from [issue #697](https://github.com/Knatte18/loomyard/issues/697).

Seen: the prime had an untracked /home/knatte/Code/loomyard-LYXHUB/loomyard/.claude/worktrees/_board/.lyx/pause/net (empty dirs, created 2026-10-15 18:03). .claude/worktrees/ is Claude Code's own directory for isolation: worktree agents, so lyx run inside such a worktree resolved the hub board dir as the sibling _board of the agent worktree, not the hub's _board, and created its pause/net state there. On 2026-10-15 the untracked directory made update-plugins.sh refuse the dirty prime, so the Land deploy of discussion-relevant-list failed (its tackler attempt diagnosed it). Workaround: /.claude/worktrees/ was added to .git/info/exclude, and the empty dirs were removed by hand on 2026-10-17. Expected: lyx run from a directory that is not a recorded fabric worktree resolves the board dir from the fabric anchor or refuses, and never creates hub state under a guessed path.