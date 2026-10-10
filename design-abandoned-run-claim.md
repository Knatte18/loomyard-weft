# An abandoned run leaves its board entry locked

`abandoned-run-claim` · Note · enhancement, batten, board

With `board-lock-running`, a board entry whose status has the run form (`<state> · <producer>`) refuses every scope change.
A run that ends without landing (stopped, abandoned, its pair removed) leaves that status behind, so the entry stays locked until someone clears the status by hand.
Seen in the PR-Gate review of PR #482.

Idea: batten's teardown of a run that did not land clears the claim (or sets a non-run status), so an abandoned task becomes editable again.
